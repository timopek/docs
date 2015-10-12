# 环信应用ELK分析客服平台错误日志

## 需求

环信客服是分布式的JAVA应用平台，在每次更新上线和日常运维中都需要通过分析错误日志定位故障。
因此我们搭建了基于ELK的日志分析平台，按照关键字过滤，收集符合条件的日志，把过滤后的日志展示
给开发和运维的同事。

先看看我们用到的ELK在官网是怎么定义的，

* Elasticsearch

  Elasticsearch是一种可高扩展的开源的全文搜索分析引擎，可以近实时的快速保存，查询和分析大
  容量的数据。

* Logstash

  Logstash是一种具备实时处理流式数据能力的开源数据收集引擎。

* Kibana

  Kibana是分析和可视化保存在Elasticsearch里的数据的开源平台。

简单点说，我们使用Logstash收集和过滤服务器上的日志，发送并保存到Elasticsearch中，再通过Kibana
展示和分析收集到的日志内容。

再来看看JAVA错误日志内容，Logstash通过多种plugin的组合实现对日志的收集，过滤，处理，转存。
能够很好的处理单行日志的同时，对于多行的日志也有很好的应对方法。
```
[ERROR][2015-09-07 00:01:11,751] com.easemob.weichat.service.IMChannelServiceImpl - jid=easemob-demo#jianguo1_1437026992conn@easemob.com login failed with 2968 times.
org.jivesoftware.smack.sasl.SASLErrorException: SASLError using PLAIN: not-authorized
        at org.jivesoftware.smack.SASLAuthentication.authenticate(SASLAuthentication.java:348) ~[kefu-connector-1.2.0.FINAL.jar:na]
        at org.jivesoftware.smack.tcp.XMPPTCPConnection.login(XMPPTCPConnection.java:244) ~[kefu-connector-1.2.0.FINAL.jar:na]
        at com.easemob.weichat.service.IMChannelServiceImpl.getXMPPConnectionAndLogin(IMChannelServiceImpl.java:312) [kefu-connector-1.2.0.FINAL.jar:na]
        at com.easemob.weichat.service.IMChannelServiceImpl.startChannel(IMChannelServiceImpl.java:249) [kefu-connector-1.2.0.FINAL.jar:na]
        at com.easemob.weichat.service.IMChannelServiceImpl.startChannel(IMChannelServiceImpl.java:89) [kefu-connector-1.2.0.FINAL.jar:na]
        at com.easemob.weichat.techchannel.easemob.IMChannelService$Processor$startChannel.getResult(IMChannelService.java:467) [kefu-connector-1.2.0.FINAL.jar:na]
        at com.easemob.weichat.techchannel.easemob.IMChannelService$Processor$startChannel.getResult(IMChannelService.java:452) [kefu-connector-1.2.0.FINAL.jar:na]
        at org.apache.thrift.ProcessFunction.process(ProcessFunction.java:39) [kefu-connector-1.2.0.FINAL.jar:na]
        at org.apache.thrift.TBaseProcessor.process(TBaseProcessor.java:39) [kefu-connector-1.2.0.FINAL.jar:na]
        at org.apache.thrift.server.AbstractNonblockingServer$FrameBuffer.invoke(AbstractNonblockingServer.java:518) [kefu-connector-1.2.0.FINAL.jar:na]
        at org.apache.thrift.server.Invocation.run(Invocation.java:18) [kefu-connector-1.2/data/apps/log/kefu/connector-error.log
```

## ELK平台架构

### 日志分析平台组成

* Logstash Shipper: 配置Logstash input plugin读取日志文件， Filter plugin处理日志,
MQ output plugin把日志写入Kafka集群

* Kafka Cluster: 作为消息队列，保存处理过的日志，作为Logstash Indxer的消息源。由于Kafka
集群的部署和本文关系不大，请自行查找部署方法。这个部分也可以用redis服务替代。

* Logstash Indxer: 配置Logstash MQ input plugin从Kafka Cluster里读取新的日志，
Filter plugin处理日志，Output plugin把日志写入Elasticsearch Cluster。

* Elasticsearch Cluster: 日志数据库。

* Log Index: Kibana从Elasticsearch Cluster读取日志并做数据可视化，Nginx配置安全访问。

![ELK架构](images/elk_chart.png "ELK架构")

## 服务配置

### Logstash Shipper

Logstash shipper和indexer只是配置不同的logstash服务，shipper负责从日志源中读取新的日志，
进行处理然后存入消息队列中。

在客服应用服务器上安装logstash
```
# yum install logstash -y
# rpm -qa | grep logstash
logstash-1.5.2-1.noarch
```
配置客服connector错误日志shipper

file: /etc/logstash/patterns/kefu.grok
```
JAVATIME %{YEAR}-%{MONTHNUM}-%{MONTHDAY}%{SPACE}%{TIME}
KEFUERROR \[%{NOTSPACE:loglevel}\]\[%{JAVATIME:logtime}\] %{GREEDYDATA:logmessage}
```
kefu.grok按照日志内容定义了grok样式，把日志从普通文本格式转换成JSON格式，这里会把一条日志
转换成三个字段，"loglevel", "logtime", "logmessage"。

file: /etc/logstash/conf.d/connector-error-shipper.conf
```
input {
  file {
    path => [ "/data/apps/log/kefu/connector-error.log" ]
    tags => [ "kefu-conn-error" ]
  }
}
filter {
  if "kefu-conn-error" in [tags] {
    multiline {
      pattern => "^\["
      negate  => true
      what    => "previous"
    }
    grok {
      patterns_dir => [ "/etc/logstash/patterns/" ]
      match => { "message" => "%{KEFUERROR}" }
      add_field => {
         "hostname"=> "kf10"
         "project"=> "kefu"
      }
      add_tag => [ "kefu_error_groked" ]
    }
    date {
      match => [ "logtime", "yyyy-MM-dd HH:mm:ss,SSS" ]
      target => "@timestamp"
    }
    if [message] =~ /java.util.ConcurrentModificationException/ {
      mutate {
        add_tag => "kefu-alert"
        add_field => { "error_type" => "java.util.ConcurrentModificationException" }
      }
    }
    if "kefu-alert" not in [tags] {
      drop { }
    }
    mutate {
      remove_field => [ "message" ]
    }
  }
}
output {
  stdout{
  }
  if "kefu-alert" in [tags] {
    kafka {
      broker_list => "kafka1:9092"
      topic_id => "kefu_webapp_error"
    }
  }
}
```
connector-error-shipper.conf文件里使用了三种类型的plugin, input, filter和output。
* input plugin
  * file: 从文件/data/apps/log/kefu/connector-error.log读取日志，增加tag "kefu-conn-error"。
* filter plugin
  * multiline: 用于处理多行日志，把多行的内容合并为一条消息。
  * grok: 根据定义样式把匹配的日志从文本格式转换成JSON格式，增加"hostname"和"project"字段，增加tag "kefu_error_groked"。
  * date: 匹配日志中的日期格式，把日期转成"@timestamp"字段，存入Elasticsearch Cluster时这个字段是日志展示的时间点。
  * mutate: 对使用正则匹配过关键字的日志，加上tag "kefu-alert"和字段"error_type"。
  * drop: 丢弃不带"kefu-alert" tag的日志。
  * mutate: 移除日志中的"message"的字段，"message"字段是日志的原始文本，日志内容已经在字段"logmessage"里，所以可以移除，减少传输开销。
* output plugin
  * stdout: 把日志写到标准输出，如果logstash服务是以service运行的，日志写入/var/log/logstash/logstash.stdout。
  * kafka: 把带有"kefu-alert" tag的日志写入"ops-ali-hangzhou-kafka1:9092"的topic "kefu_webapp_error"。

在应用服务器上启动logstash服务后，新的日志经过处理后将存到kafka集群的topic kefu_webapp_error里。
```
# service logstash start
```

### Logstash Indexer

在logindex服务器上安装logstash
```
# yum install logstash -y
# rpm -qa | grep logstash
logstash-1.5.2-1.noarch
```
配置Logstash Indexer

file: /etc/logstash/conf.d/logstash-indexer.conf
```
input {
  kafka {
    zk_connect => "zk1:2181"
    topic_id => "kefu_webapp_error"
    tags => "kefu_webapp_error"
  }
}
output {
  if "kefu_webapp_error" in [tags] {
    stdout { }
    elasticsearch {
      index => "kefu_webapp_error-%{+YYYY.MM.dd}"
      host => ["ela-data1:9200", "ela-data2:9200", "ela-data3:9200"]
      protocol => "http"
    }
  }
}
```
logstash-indexer.conf文件里使用了两种类型的plugin, input和output。
* input plugin
  * kafka: 指定zookeeper的连接字符串"zk1:2181"和Kafka topic "kefu_webapp_error"，增加tag "kefu_webapp_error"
* output plugin
  * stdout: 把日志写到标准输出，如果logstash服务是以service运行的，日志写入/var/log/logstash/logstash.stdout。
  * elasticsearch: 把带有"kefu_webapp_error" tag的日志写入"logindex:9200"的index "kefu_webapp_error-%{+YYYY.MM.dd}"，以http的协议连接elasticsearch。

在logindex服务器上启动logstash服务后，kafka集群的topic kefu_webapp_error里的日志将被传
送到elasticsearch集群里，保存在以kefu_webapp_error为前缀命名的index里。
```
# service logstash start
```

### Elasticsearch Cluster

Elasticsearch可以配置两种角色，data节点和master节点，这两种角色的区别可以简单的理解为，
data节点保存数据，master节点不保存数据。master节点可以实现elasticsearch集群的读写分
离。

在架构图里，我们配置了四个节点，三个data节点和一个master节点。master节点是运行在logindex
服务器上作为读数据接口，三个数据节点配置SLB，SLB的地址作为写数据接口。SLB非必须，也可以把
三个data节点的列表作为写数据接口。

配置数据节点/etc/elasticsearch/elasticsearch.yml, 配置节点2和3时，修改node.name。
```
cluster.name: OPS_HZ_LOG_Cluster
node.name: ela-data1
node.master: true
node.data: true
bootstrap.mlockall: true
index.cache.field.max_size: 500000
index.cache.field.expire: 5m
discovery.zen.fd.ping_timeout: 60s
discovery.zen.fd.ping_retries: 10
discovery.zen.fd.ping_interval: 5s
discovery.zen.ping.unicast.hosts: [ "ela-data1:9300", "ela-data2:9300", "ela-data3:9300", "logindex:9300" ]
http.enabled: true
http.jsonp.enable: true
```
配置master节点，/etc/elasticsearch/elasticsearch.yml
```
cluster.name: OPS_HZ_LOG_Cluster
node.name: ops-ali-hangzhou-logindex
node.master: true
node.data: false
bootstrap.mlockall: true
index.cache.field.max_size: 500000
index.cache.field.expire: 5m
discovery.zen.fd.ping_timeout: 60s
discovery.zen.fd.ping_retries: 10
discovery.zen.fd.ping_interval: 5s
discovery.zen.ping.unicast.hosts: [ "ela-data1:9300", "ela-data2:9300", "ela-data3:9300", "logindex:9300" ]
http.enabled: True
http.jsonp.enable: True
```
配置文件修改后，重启服务
```
# service elasticsearch restart
```

查看集群状态
```
# curl localhost:9200/_cluster/health?pretty
{
  "cluster_name" : "OPS_HZ_LOG_Cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 4,
  "number_of_data_nodes" : 3,
  "active_primary_shards" : 431,
  "active_shards" : 862,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0
}

# curl localhost:9200/_cat/nodes?pretty
logindex      10.16.251.166 14 46 1.52 - m logindex
ela-data3     10.16.120.77  23 72 0.00 d m ela-data3
ela-data2     10.16.112.227 28 73 0.00 d * ela-data2
ela-data1     10.16.117.74  20 73 0.00 d m ela-data1
```

### Kibana
把kibana-4.1.1下载解压到/data/kibana, 启动服务, 查看端口5601是否在侦听。
```
# cd /data/kibana
# ./bin/kibana >/dev/null &
# netstat -lnt | grep 5601
```
配置Nginx，使用账户密码访问kibana。配置文件/etc/nginx/conf.d/kibana.conf
```
upstream kibana
{
    server 127.0.0.1:5601 max_fails=50 weight=3;
}

server {
    listen       80;
    listen       443;
    server_name  _;

    ssl                  on;
    ssl_certificate      /etc/nginx/https/server.crt;
    ssl_certificate_key  /etc/nginx/https/server.key;

    ssl_session_timeout  5m;

    ssl_protocols  SSLv2 SSLv3 TLSv1;
    ssl_ciphers  ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
    ssl_prefer_server_ciphers   on;

    location / {
        auth_basic "Password Required";
        auth_basic_user_file /etc/nginx/htpasswd/easemob.pwd;
        proxy_pass http://kibana;
    }
}
```
制作htpasswd密码文件
```
# yum install httpd-tools
# htpasswd -bc easemob.pwd username password
# mv easemob.pwd /etc/nginx/htpasswd/
```
启动Nginx
```
# service nginx restart
```
通过浏览器访问kibana的地址https://10.16.251.166/ 设置index是“kefu_webapp_error-*”。
![kibana_index](images/kibana01.png "kibana index")

在Discovery页面下可以看到收集到的日志了。
![kibana_show](images/kibana02.png "kibana show")

##关于作者

窦中强
环信运维团队devops工程师。
