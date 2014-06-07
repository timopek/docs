Redis cluster
=============

Nodes
-----

Master: 172.205.0.9:6379, 172.205.0.2:6379, 172.205.0.14:6379

Slave: 172.205.0.15:6379, 172.205.0.15:6380, 172.205.0.15:6381

* 172.205.0.9
  conf: /opt/redis/redis.conf
  log: /opt/redis/redis.log

* 172.205.0.2
  conf: /opt/redis/redis.conf
  log: /opt/redis/redis.log

* 172.205.0.14
  conf: /opt/redis/redis.conf
  log: /opt/redis/redis.log

* 172.205.0.15
  conf: /opt/redis{1,2,3}/redis.conf
  log: /opt/redis{1,2,3}/redis.log


Install
--------

    # wget https://github.com/antirez/redis/archive/3.0.0-beta1.tar.gz
    # tar zxf 3.0.0-beta1
    # cd redis-3.0.0-beta1/deps
    # make hiredis jemalloc linenoise lua
    # cd ..
    # make && make install

Conf
--------

file: /opt/redis/redis.conf

    daemonize yes
    pidfile "/var/run/redis.pid"
    port 6379
    tcp-backlog 511
    timeout 0
    tcp-keepalive 0
    loglevel notice
    logfile ""
    databases 16
    slave-serve-stale-data yes
    slave-read-only no
    repl-disable-tcp-nodelay no
    slave-priority 100
    maxmemory 100mb
    maxmemory-policy allkeys-lru
    lua-time-limit 5000
    notify-keyspace-events ""
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-node-timeout 5000
    appendonly yes

file: /etc/sysctl.conf, add the line "vm.overcommit_memory = 1" and run
    
    # sysctl -p

    
Start
-----

    172.205.0.9:
    # /usr/local/bin/redis-server /opt/redis/redis.conf
    172.205.0.2:
    # /usr/local/bin/redis-server /opt/redis/redis.conf 
    172.205.0.14:
    # /usr/local/bin/redis-server /opt/redis/redis.conf 
    172.205.0.15:
    # /usr/local/bin/redis-server /opt/redis1/redis.conf
    # /usr/local/bin/redis-server /opt/redis2/redis.conf 
    # /usr/local/bin/redis-server /opt/redis3/redis.conf 


Create Cluster
--------------

Install ruby on control node

    # yum install openssl openssl-devel -y

    Install ruby and gem install redis

    # ./src/redis-trib.rb create --replicas 1 172.205.0.9:6379 172.205.0.2:6379 172.205.0.14:6379 172.205.0.15:6379 172.205.0.15:6380 172.205.0.15:6381
