Fluentd
=======

Install Ruby
------------

    # wget http://cache.ruby-lang.org/pub/ruby/2.1/ruby-2.1.2.tar.gz
    # tar zxf ruby-2.1.2.tar.gz
    # cd ruby-2.1.2
    # ./configure
    # make
    # make install
    # ruby -v

Install fluentd
---------------

    # gem install fluentd
    # fluentd --setup /opt/fluent
    # chown -R easemob.easemob /opt/fluent
    Use taobao gem source
    # gem sources --remove https://rubygems.org/
    # gem sources -a https://ruby.taobao.org/
    # gem sources -l
    Create the log dir
    # mkdir /var/easemob/fluentd
    # chown -R easemob.easemob /var/easemob/fluentd

Syslog to Elasticsearch
-----------------------

http://docs.fluentd.org/recipe/syslog/elasticsearch

    # gem install fluent-plugin-elasticsearch
    
Add the line "include conf.d/*.conf" into file /opt/fluent/fluent.conf
    
write the conf to /opt/conf.d/syslog.conf

    <source>
      type syslog
      port 42185
      tag syslog
    </source>
    <match syslog.**>
       type copy
       <store>
          type file
          path /var/log/fluentd/fluentd-syslog.log
       </store>
       <store>
          type elasticsearch
          logstash_format true
          flush_interval 10s
          host 127.0.0.1 # elasticsearch host
          port 9200 
          index_name fluentd_easemob  
          type_name hostname #change as the hostname
       </store>
     </match>

start the fluentd

    # fluentd -c /opt/fluent/fluent.conf


Nginx to Elasticsearch
-----------------------

