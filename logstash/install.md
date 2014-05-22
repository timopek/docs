Logstash

* Shipper: Sends events to LogStash. Your remote agents will
generally only run this component.

* Broker and Indexer: Receives and indexes the events.

* Search and Storage: Allows you to search and store events.

* Web Interface: A Web-based interface to LogStash (there are
two flavors we'll look at).

Basic
1. Java.
2. Get logstash
   RPM: http://logstash.net/docs/1.4.1/repositories
   TAR: https://download.elasticsearch.org/logstash/logstash/logstash-1.4.1.tar.gz
3. logstash.conf
   There are 3 conf blocks
   * inputs - How events get into LogStash.
   * filters - How you can manipulate events in LogStash.
   * outputs - How you can output events from LogStash.
   ./bin/logstash agent -f conf/sampble.conf

Plan:
1. Build a single central logstash server.
2. Configure your central server to receive events, index them and make them avaliable to search.
3. Install logstash on a remote agent.
4. Configure Logstash to send some selected log events from our remote agent to our central server.
5. Install the logstash Kibana agent to act as a web console.

Event Lifesycle
* The logstash agent on our remote agents collects and sends a log event to our central server.
* A Redis instance receives the log event on the central server and acts as a buffer
* The logstash agent draws the log event from our Redis instance and indexes it.
* The logstash agent sends the indexed event to ELasticSearch.
* ElasticSearch stores and renders the event searchable.
* The Logstash web interface queries the event from ElasticSearch

Installing Logstash on the central server
1. Central Server:
10.66.129.39 dhcp-129-39.nay.redhat.com
a. Download the tar file and unzip it in /opt/logstash
b. Create the directory. logstash/conf logstash/logs

2. Broker
a. install redis
  # yum install redis
b. Change the redis interface
  # vim /etc/redis.conf
  bind 10.66.129.39
c. Test the Redis running
  # redis-cli -h 10.66.129.39

3. ElasticSearch
The ElasticSearch server version needs
to match the version of the ElasticSearch client that is bundled with
LogStash.
http://logstash.net/docs/latest/outputs/elasticsearch

RPM:
https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.1.1.noarch.rpm

Install and configure
Modify the file /etc/elasticsearch/elasticsearch.yml
/etc/init.d/elasticsearch start
Enable the port 9200
Check: http://10.66.129.39:9200/
http://10.66.129.39:9200/_status?pretty=true

4. Creating a basic central configuration
[root@dhcp-129-39 conf]# cat central.conf 
input {
      redis {
          host => "10.66.129.39"
          type => "redis-input"
          data_type => "list"
          key => "easemob"
      }
}
output {
      stdout { }
      elasticsearch {
          cluster => "easemob"
      }
}

# ./bin/logstash agent -f conf/central.conf web

5. Installing logstash on our first agent
10.66.129.40 dhcp-129-40.nay.redhat.com
[root@dhcp-129-40 conf]# cat shipper.conf
input {
      file {
          type => "syslog"
          path => ["/var/log/secure","/var/log/messages"]
          exclude => ["*.gz", "shipper.log"]
      }
}
output {
      stdout { }
      redis {
          host => "10.66.129.39"
          data_type => "list"
          key => "easemob"
       }
}

# ./bin/logstash -f conf/shipper.conf 

