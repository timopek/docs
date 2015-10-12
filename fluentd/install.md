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
    # fluentd --setup /opt/fluentd
    # chown -R easemob.easemob /opt/fluentd
    Use taobao gem source
    # gem sources --remove https://rubygems.org/
    # gem sources -a https://ruby.taobao.org/
    # gem sources -l
    Create the log dir
    # mkdir /var/easemob/fluentd
    # chown -R easemob.easemob /var/easemob/fluentd
    # cd /var/easemob/fluentd
    # mkdir cassandra ejabberd kafka nginx tomcat
	
	# gem install fluent-plugin-record-modifier

Configure fluentd in supervisor
-------------------------------

    [program:fluentd]
    command=/usr/local/bin/fluentd -c /opt/fluentd/fluent.conf
    process_name=fluentd
    user=easemob
    autostart=true
    autorestart=true
    startsecs=10
    startretries=999
    log_stdout=true
    log_stderr=true
    logfile=/var/easemob/fluentd/fluentd.out
    logfile_maxbytes=20MB
    logfile_backups=10

Syslog to Elasticsearch
-----------------------

http://docs.fluentd.org/recipe/syslog/elasticsearch

    # sudo yum install libcurl libcurl-devel -y
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

    # fluentd -c /opt/fluentd/fluent.conf


Nginx to Elasticsearch
-----------------------


Tomcat to Elasticsearch
-----------------------

Log Filter
----------
    
    # sudo yum install libcurl libcurl-devel -y
    # gem install fluent-plugin-record-modifier


1. https://github.com/repeatedly/fluent-plugin-record-modifier

    <match filter.nginx>
        type record_modifier
        tag filter
        hostname ebs-ali-beijing-3
    </match>
    <match filter>
        type copy
        <store>
           type file
           path /var/easemob/fluentd/fluentd-nginx.log
        </store>
        <store>
            type elasticsearch
            logstash_format true
            host 223.202.120.59
            port 9200
            index_name fluentd_easemob
            type_name nginx
            flush_interval 10s
        </store>
    </match>

fluent-plugin-grep
-------------------

    $ sudo gem install fluent-plugin-grep

