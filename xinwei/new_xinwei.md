1. 安装前的准备工作
-----------------

###1.1 安装需要的软件包###

####安装epel源####

    yum install http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm -y

####基础包####

    yum install libyaml libyaml-devel openssl openssl-devel \
    zlib zlib-devel ImageMagick lsof htop systat iptraf-ng \
    wget git curl htop iftop iotop nmap screen libseliunx-python -y

####Build包####

    yum install autoconf automake binutils bison flex gcc gcc-c++ \
    gettext libtool make patch pkgconfig redhat-rpm-config rpm-build -y

####Python包####

    yum install Cython libevent libevent-devel python python-devel \
    python-setuptools python-pip -y

####pip安装python包####

    pip install requests virtualenv redis httplib2

###1.2 系统更新###

    yum upgrade -y
    init 6


###1.3 创建用户###

我们需要创建一个普通用户组和用户easemob, 家目录是/home/easemob

    useradd -d /home/easemob easemob
    password easemob

并且给它sudo的权限

    echo "easemob ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

###1.4 创建需要的目录###

创建应用安装的目录

    mkdir /home/easemob/apps/{data,opt,var,log,config} -p
    chown -R easemob.easemob /home/easemob/apps

###1.5 设置时区和时间同步###

####安装时间同步的包####

    yum install tzdata ntp ntpdate -y

####设置ntp####

    service ntpd restart

###1.6 安装JDK###

首先要先卸载openjdk, 

    yum remove *openjdk*

然后安装上官方的JDK，

    wget http://www.easemob.com/downloads/install/jdk-7u25-linux-x64.tar.gz -P /tmp
    mkdir /usr/local/java -p
    tar zxf /tmp/jdk-7u25-linux-x64.tar.gz -C /usr/local/java
    ln -s /usr/local/java/jdk1.7.0_25 /usr/local/java/jdk
    echo "export JAVA_HOME=/usr/local/java/jdk" >> /etc/profile
    echo "PATH=$PATH:$HOME/bin:'$JAVA_HOME/bin'" >> /etc/profile
    source /etc/profile
    
检查JDK Version

    java -version

###1.7 系统优化###

把下面的内容追加到文件/etc/sysctl.cof, 然后运行命令`sysctl -p`

    vm.swappiness = 0
    net.ipv4.neigh.default.gc_stale_time = 120
    net.ipv4.conf.all.rp_filter = 0
    net.ipv4.conf.default.rp_filter = 0
    net.ipv4.conf.default.arp_announce = 2
    net.ipv4.conf.all.arp_announce = 2
    net.ipv4.tcp_max_tw_buckets = 20000
    net.ipv4.tcp_syncookies = 1
    net.ipv4.tcp_max_syn_backlog = 8192
    net.ipv4.tcp_synack_retries = 2
    net.ipv4.conf.lo.arp_announce = 2
    fs.file-max = 999999
    net.ipv4.ip_conntrack_max = 999999
    net.ipv4.netfilter.ip_conntrack_max = 999999
    net.ipv4.tcp_keepalive_time = 300
    net.ipv4.tcp_keepalive_probes = 10
    net.ipv4.tcp_keepalive_intvl = 30
    net.ipv6.conf.default.disable_ipv6 = 1
    net.ipv6.conf.all.disable_ipv6 = 1
    net.core.rmem_max = 16777216
    net.core.wmem_max = 16777216
    net.ipv4.tcp_rmem = 4096 87380 16777216
    net.ipv4.tcp_wmem = 4096 87380 16777216
    net.ipv4.tcp_mem = 50576 64768 98152
    net.core.netdev_max_backlog = 2048
    net.core.somaxconn = 2048
    net.ipv4.tcp_tw_reuse = 1
    net.ipv4.tcp_tw_recycle = 1
    net.ipv4.tcp_fin_timeout = 30
    net.ipv4.ip_local_port_range = 1024 65000
    vm.overcommit_memory = 1
    net.ipv4.tcp_slow_start_after_idle = 0
    net.ipv4.tcp_no_metrics_save = 1
    vm.max_map_count = 131072

把下面的内容追加到文件/etc/security/limits.conf, 并退出终端重新登陆服务器。

    easemob       hard    core            0
    easemob       soft    core            0
    easemob       hard    data            unlimited
    easemob       soft    data            unlimited
    easemob       hard    fsize           unlimited
    easemob       soft    fsize           unlimited
    easemob       hard    memlock         unlimited
    easemob       soft    memlock         unlimited
    easemob       hard    nofile          999999
    easemob       soft    nofile          999999
    easemob       hard    rss             unlimited
    easemob       soft    rss             unlimited
    easemob       hard    stack           unlimited
    easemob       soft    stack           unlimited
    easemob       hard    cpu             unlimited
    easemob       soft    cpu             unlimited
    easemob       hard    nproc           unlimited
    easemob       soft    nproc           unlimited
    easemob       hard    as              unlimited
    easemob       soft    as              unlimited
    easemob       hard    locks           unlimited
    easemob       soft    locks           unlimited
    easemob       hard    sigpending      unlimited
    easemob       soft    sigpending      unlimited
    easemob       hard    msgqueue        unlimited
    easemob       soft    msgqueue        unlimited
    easemob       hard    rtprio          unlimited
    easemob       soft    rtprio          unlimited


###1.8 安装supervisor###

安装包

    pip install supervisor
    mkdir /home/easemob/apps/{data,var,log,config}/supervisor -p
    chown -R easemob.easemob /home/easemob/apps

配置supervisor

    echo_supervisord_conf > /home/easemob/apps/config/supervisor/supervisord.conf
    chmod 640 /home/easemob/apps/config/supervisor/supervisord.conf
    chown easemob.easemob /home/easemob/apps/config/supervisor/supervisord.conf

编辑配置文件/home/easemob/apps/config/supervisor/supervisord.conf

    [unix_http_server]
    file=/home/easemob/apps/var/supervisor/supervisor.sock
    chown=easemob:easemob
    [inet_http_server]
    port={{hostname}}:9001 ；请自行以实际的域名替换{{hostname}}
    [supervisord]
    logfile=/home/easemob/apps/log/supervisor/supervisord.log
    logfile_maxbytes=10MB
    logfile_backups=10
    loglevel=trace
    pidfile=/home/easemob/apps/var/supervisor/supervisord.pid 
    childlogdir=/home/easemob/apps/log/supervisor/
    nodaemon=false 
    user=easemob
    nocleanup=true
    minfds = 1024
    minprocs = 200
    [rpcinterface:supervisor]
    supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
    [supervisorctl]
    serverurl=unix://home/easemob/apps/var/supervisor/supervisor.sock
    [include]
    files = /home/easemob/apps/data/supervisor/*.conf

设置自动启动supervisor, 复制这个[supervisor脚本](http://www.easemob.com/downloads/install/conf/supervisor/etc_init.d_supervisor.j2)的代码到文件/etc/init.d/supervisor

    service supervisor start
    chkconfig supervisor on

配置脚本env.sh, 把下面这以行写入文件/home/easemob/apps/config/env.sh

    alias supervisorctl='supervisorctl -c /home/easemob/apps/config/supervisor/supervisord.conf'


###1.9 设置域名###

给服务器设置域名，编辑文件/etc/sysconfig/network，设置HOSTNAME=$hostname，然后运行命令`hostname $hostname`, 自行将$hostname替换为实际域名.

编辑文件/etc/hosts, 追加一行，自行将$IP和$HOSTNAME替换为实际IP地址和域名.

    $IP $HOSTNAME


2. 安装Cassandra
-----------------------

在easemob的用户下安装配置cassandra

2.1 创建相关目录

    mkdir /home/easemob/apps/{data,var,log,config}/cassandra -p
    mkdir /home/easemob/apps/data/cassandra/{data,commitlog,save_caches} -p
    chown -R easemob.easemob /home/easemob/apps/

2.2 安装依赖的包

    yum install boost-devel jna numactl numactl-devel -y

2.3 下载[cassandra压缩文件](http://archive.apache.org/dist/cassandra/2.0.7/apache-cassandra-2.0.7-bin.tar.gz) 到目录/tmp, 解压缩文件，并移动解压后的目录apache-cassandra-2.0.7到目录/home/easemob/apps/opt。最后创建软链接。

    ln -s /home/easemob/apps/opt/apache-cassandra-2.0.7 /home/easemob/apps/opt/cassandra
    chown -R easemob.easemob /home/easemob/apps/

2.4 配置cassandra

复制配置文件

    cp /home/easemob/apps/opt/cassandra/cassandra.yaml /home/easemob/apps/config/cassandra
    cp /home/easemob/apps/opt/cassandra/cassandra-env.sh /home/easemob/apps/config/cassandra
    cp /home/easemob/apps/opt/cassandra/log4j-server.properties /home/easemob/apps/config/cassandra

修改配置文件/home/easemob/apps/config/cassandra/cassandra.yaml,
   - 找到"data_file_directories", 把它的下一行改为/home/easemob/apps/data/cassandra/data
   - 找到"commitlog_directory", 把它后面的路径改为/home/easemob/apps/data/cassandra/commitlog
   - 找到"saved_caches_directory", 把它后面的路径改为/home/easemob/apps/data/cassandra/saved_caches
   - 找到"seeds", 把它后面的值改成服务器的域名或IP地址
   - 找到"listen_address", 把它后面的值改成服务器的域名或IP地址
   - 找到"rpc_address", 把它后面的值改成服务器的域名或IP地址

修改配置文件/home/easemob/apps/config/cassandra/log4j-server.properties,
   - 找到"data_file_directories", 把它的下一行改为/home/easemob/apps/data/cassandra/data

把下面的内容写到文件/home/easemob/apps/config/cassandra/cassandra-in.sh

    CASSANDRA_HOME=/home/easemob/apps/opt/cassandra
    CASSANDRA_CONF=/home/easemob/apps/config/cassandra

    cassandra_bin="$CASSANDRA_HOME/build/classes/main"
    cassandra_bin="$cassandra_bin:$CASSANDRA_HOME/build/classes/thrift"
    JAVA_HOME=/usr/local/java/jdk

    CLASSPATH="$CASSANDRA_CONF:$cassandra_bin"

    for jar in "$CASSANDRA_HOME"/lib/*.jar; do
        CLASSPATH="$CLASSPATH:$jar"
    done
 
    if [ "$JVM_VENDOR" != "OpenJDK" -o "$JVM_VERSION" \> "1.6.0" ] \
        || [ "$JVM_VERSION" = "1.6.0" -a "$JVM_PATCH_VERSION" -ge 23 ]
    then
        JAVA_AGENT="$JAVA_AGENT -javaagent:$CASSANDRA_HOME/lib/jamm-0.2.5.jar"
    fi

配置cron任务，把命令`/home/easemob/apps//opt/cassandra/bin/nodetool repair -par`设置成每天执行一次。

2.5 配置supervisord管理cassandra服务

写入配置文件/home/easemob/apps/data/supervisor/cassandra.conf,

    [program:cassandra]
    command=/home/easemob/apps/opt/cassandra/bin/cassandra -f -p /home/easemob/apps/var/cassandra/cassandra.pid
    user=easemob
    autostart=true
    autorestart=true
    startsecs=10
    startretries=999
    log_stdout=true
    log_stderr=true
    logfile=/home/easemob/apps/log/cassandra/cassandra.out
    logfile_maxbytes=20MB
    logfile_backups=10
    environment=CASSANDRA_INCLUDE="/home/easemob/apps/config/cassandra/cassandra-in.sh"

3. 安装Zookeeper
-----------------

3.1 创建目录

    mkdir /home/easemob/apps/{data,var,log,config}/zookeeper -p
    
3.2 下载[zookeeper](http://mirrors.cnnic.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz), 解压到目录/home/easemob/apps/opt, 并创建软链接，

    ln -s /home/easemob/apps/opt/zookeeper-3.4.6 /home/easemob/apps/opt/zookeeper
    chown -R easemob.easemob /home/easemob/apps/

3.3 配置zookeeper

复制文件/home/easemob/apps/zookeeper/conf/zoo_sample.cfg到/home/easemob/apps/config/zookeeper/zoo.cfg
 - 替换"dataDir=/tmp/zookeeper"为"dataDir=/home/easemob/apps/data/zookeeper"

复制文件/home/easemob/apps/zookeeper/conf/log4j.properties到/home/easemob/apps/config/zookeeper/log4j.properties

复制文件/home/easemob/apps/zookeeper/conf/configuration.xsl到/home/easemob/apps/config/zookeeper/configuration.xsl

新建文件/home/easemob/apps/data/zookeeper/myid，文件内容是"1"

3.4 配置supervisord管理zookeeper

写入配置文件/home/easemob/apps/data/supervisor/zookeeper.conf,

    [program:zookeeper]
    command=/home/easemob/apps/opt/zookeeper/bin/zkServer.sh start-foreground
    user=easemob
    autostart=true
    autorestart=true
    startsecs=10
    startretries=999
    log_stdout=true
    log_stderr=true
    logfile=/home/easemob/apps/log/zookeeper/zookeeper.out
    logfile_maxbytes=20MB
    logfile_backups=10
    environment=JMXDISABLE="true",ZOOCFGDIR="/home/easemob/apps/config/zookeeper",ZOO_LOG_DIR="/home/easemob/apps/log/zookeeper",ZOOPIDFILE="/home/easemob/apps/var/zookeeper/zookeeper.pid"


4. 安装Nginx
-----------------

4.1 创建目录

    mkdir /home/easemob/apps/{data,var,log,config}/nginx -p
    mkdir /home/easemob/apps/config/nginx/conf.d -p
    chown -R easemob.easemob /home/easemob/apps/

4.2 安装依赖

    yum install git pcre pcre-devel pcre-static openssl zlib zlib-devel gzip snappy -y

4.3 安装nginx

下载[nginx-1.7.0](http://www.easemob.com/downloads/install/nginx-1.7.0.tar.gz)，并解压到目录/tmp

安装运行nginx
    
    cd /tmp/nginx-1.7.0
    ./configure --with-http_ssl_module --with-http_realip_module \
    --with-http_addition_module --with-http_sub_module \
    --with-http_dav_module --with-http_flv_module --with-http_mp4_module \
    --with-http_gunzip_module --with-http_gzip_static_module \
    --with-http_random_index_module --with-http_secure_link_module \
    --with-http_stub_status_module --with-file-aio \
    --with-cc-opt='-O2 -g -pipe -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector \
    --param=ssp-buffer-size=4 -m64 -mtune=generic' --without-mail_pop3_module \
    --without-mail_imap_module --without-mail_smtp_module \
    --prefix=/home/easemob/apps/opt/nginx-1.7.0 \
    --conf-path=/home/easemob/apps/config/nginx/nginx.conf \
    --user=easemob --group=easemob \
    --pid-path=/home/easemob/apps/var/nginx/nginx.pid \
    --error-log-path=/home/easemob/apps/log/nginx/error.log \
    --http-log-path=/home/easemob/apps/log/nginx/access.log \
    --sbin-path=/home/easemob/apps/opt/nginx-1.7.0/sbin/nginx \
    --lock-path=/home/easemob/apps/var/nginx/nginx.lock \
    --http-client-body-temp-path=/home/easemob/apps/var/nginx/client_temp  \
    --http-proxy-temp-path=/home/easemob/apps/var/nginx/proxy_temp  \
    --http-fastcgi-temp-path=/home/easemob/apps/var/nginx/fastcgi_temp  \
    --http-uwsgi-temp-path=/home/easemob/apps/var/nginx/uwsgi_temp  \
    --http-scgi-temp-path=/home/easemob/apps/var/nginx/scgi_temp

    make
    make install

    ln -s /home/easemob/apps/opt/nginx-1.7.0 /home/easemob/apps/opt/nginx
    chown -R easemob.easemob /home/easemob/apps/
    cp /tmp/nginx-1.7.0/script/nginx /etc/init.d/
    /etc/init.d/nginx start
    chkconfig nginx on


5. 安装Redis
-----------------

5.1 创建目录

    mkdir /home/easemob/apps/{data,var,log,config}/redis -p
    chown -R easemob.easemob /home/easemob/apps/

5.2 下载[redis](http://download.redis.io/redis-stable.tar.gz)到/tmp，并解压

5.3 安装

    cd /tmp/redis-stable
    make
    make PREFIX="/home/easemob/apps/opt/redis" install

5.4 配置

复制/tmp/redis-stable/redis.conf到目录/home/easemob/apps/config/redis/redis.conf
- 找到"# bind 127.0.0.1" 替换为"bind $IP", $IP为域名或IP地址
- 找到"logfile"开头的那一行，在引号里加上路径"/home/easemob/apps/log/redis/server.log"
- 找到"dir ./"开头的那一行，替换为"dir /home/easemob/apps/data/redis"

5.5 配置supervisord管理redis

写入配置文件/home/easemob/apps/data/supervisor/redis.conf,

    [program:redis]
    command = /home/easemob/apps/opt/redis/bin/redis-server /home/easemob/apps/config/redis/redis.conf
    autostart=true   
    autorestart=true 
    startsecs=10
    user=easemob
    startretries=999
    log_stdout=true
    log_stderr=true
    logfile=/home/easemob/apps/log/redis/redis-server.out
    logfile_maxbytes=20MB


6. 安装Usergrid
-----------------

3.1 下载[tomcat 7.0.54](http://www.easemob.com/downloads/install/apache-tomcat-7.0.54.tar.gz)压缩包并解压到/home/easemob/apps/opt

创建软链接, 然后删除webapps目录下的所有文件, 下载[ROOT.war]

    ln -s /home/easemob/apps/opt/apache-tomcat-7.0.54 /home/easemob/apps/opt/tomcat
    rm -rf /home/easemob/apps/opt/tomcat
    wget http://www.easemob.com/downloads/install/ROOT.war -P /home/easemob/apps/opt/tomcat/webapps
    chown -R easemob.easemob /home/easemob/apps/


3.2 配置usergrid 

修改配置文件/home/easemob/apps/opt/tomcat/conf/server.xml, 找到localhost都替换为域名或IP地址

修改配置文件/opt/tomcat-usergrid/lib/usergrid-custom.properties,  找到localhost都替换为域名或IP地址

配置usergrid的nginx，替换localhost为IP或域名

    upstream tomcatcluster
     {
        server localhost:8080;
     }

    server
     {
        # listen 443 default_server ssl;
        listen 80;
        charset UTF-8;
        server_name localhost;
        index index.html index.htm index.jsp index.do default.jsp default.do index.php;
        root  /home/easemob/apps/data/nginx/www;
        proxy_connect_timeout 1000;
        proxy_read_timeout 1000;
        proxy_send_timeout 1000;
        access_log /home/easemob/apps/log/nginx/usergrid-access.log main;
        error_log /home/easemob/apps/log/nginx/usergrid-error.log;

        location / {
            proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass http://tomcatcluster;
          }
      }


3.3 配置supervisord管理usergrid服务

写入配置文件/home/easemob/apps/data/tomcat,

    [program:tomcat]
    command=/home/easemob/apps/opt/tomcat/bin/catalina.sh run
    user=easemob
    autostart=true
    autorestart=true
    startsecs=10
    startretries=999
    log_stdout=true
    log_stderr=true
    logfile=/home/easemob/apps/log/tomcat/tomcat.out
    logfile_maxbytes=20MB
    logfile_backups=10

3.4 按装后的初始化工作
-------------------------------

初始化数据库, 自行将localhost替换为域名
    
    # curl --user "webmaster:1234567890" http://localhost:8080/system/database/setup
    {
      "action" : "cassandra setup",
      "status" : "ok",
      "timestamp" : 1404545062339,
      "duration" : 71
    }
    # curl --user "webmaster:1234567890" http://localhost:8080/system/superuser/setup
    {
       "action" : "superuser setup",
       "status" : "ok",
       "timestamp" : 1404545009469,
       "duration" : 2
    }
    


7. 安装Cloudcode
------------------------

参考usergrid的安装，只需要修改server.xml里的端口80005，8080，8009，不要和usergrid冲突

      <Server port="8005" shutdown="SHUTDOWN"> 
     <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
      <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />


8. 安装Ejabberd
----------------------

    1)下载[ejabberd-13.12](http://www.process-one.net/downloads/ejabberd/13.12/ejabberd-13.12-linux-x86_64-installer.run)安装文件到/目录
      # wget http://www.process-one.net/downloads/ejabberd/13.12/ejabberd-13.12-linux-x86_64-installer.run -p /tmp
    2)安装
      # cd /tmp
      # chmod a+x ejabberd-13.12-linux-x86_64-installer.run
      # ./ejabberd-13.12-linux-x86_64-installer.run
      select language: 2-English(default)
      Installation Directory : /home/easemob/apps/opt/ejabberd-13.12
      domain: 这个设置成域名的域
      admin: admin
      password: thepushbox
      Cluster: no
      
    3)安装完第一台后，把文件~/.erlang.cookie复制到另外机器上，然后安装ejabberd。（单机安装这一步不需要）
    4)在服务器上修改配置文件/home/easemob/apps/opt/ejabberd-13.12/conf/ejabberd.cfg
      修改配置文件
      
      **conf/ejabberd.conf**
      加入下面两行，
      """
      {auth_method, external}.
      {extauth_program, "/home/easemob/apps/opt/ejabberd-13.12/scripts/external_auth.py"}.
      """
      还有在modules这个位置像下面这样加入mod_send_ack和mod_ping，
      {modules,
      [
        {mod_adhoc,    []},
        {mod_send_ack, []},
        {mod_ping, [ ]},
      """
        
      **conf/inetrc**
      修改host的那一行，用实际的IP和域名替换，注意IP地址是用,分割的。
      {host,{192,168,1,13}, ["static-1-13","localhost"]}.
      
      **bin/ejabberdctl**
      修改下面这两行，用实际的IP和域名替换。
      INET_DIST_INTERFACE="192.168.1.13"
      ERLANG_NODE=ejabberd@static-1-13
   
    5)单机不需要做这个, 下载[easy_cluster.beam](http://www.easemob.com/downloads/ebin.zip)解压后复制到目录/home/easemob/apps/opt/ejabberd-13.12/lib/ejabberd-13.12/ebin/
    6)在所有服务器上启动ejabberd, 启动后5222端口将被侦听，并把这个命令设置为开机启动。
      # /home/easemob/apps/opt/ejabberd-13.12/bin/ejabberdctl start
    7)把所有服务器做成ejabberd集群，单机不需要做这个。
      # /opt/ejabberd-13.12/bin/ejabberdctl debug
      (ejabberd@static-1-13)1> easy_cluster:test_node('ejabberd@static-1-12').
      server is reached.
      ok
      (ejabberd@static-1-13)2>  easy_cluster:join_as_master('ejabberd@static-1-12').
      ok
      两个ctrl+c退出ejabberd console.


9. 启动服务

    /etc/init.d/nginx restart
    /etc/init.d/supervisor restart
