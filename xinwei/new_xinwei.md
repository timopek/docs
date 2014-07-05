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


3. 安装Usergrid
-----------------


4. 安装Cloudcode
------------------------


5. 安装Nginx
-----------------


6. 安装Zookeeper
-----------------


7. 安装Redis
-----------------

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
   
    5)下载[easy_cluster.beam](http://www.easemob.com/downloads/ebin.zip)解压后复制到目录/home/easemob/apps/opt/ejabberd-13.12/lib/ejabberd-13.12/ebin/
    6)在所有服务器上启动ejabberd, 启动后5222端口将被侦听
      # /home/easemob/apps/opt/ejabberd-13.12/bin/ejabberdctl start
    7)把所有服务器做成ejabberd集群，单机不需要做这个。
      # /opt/ejabberd-13.12/bin/ejabberdctl debug
      (ejabberd@static-1-13)1> easy_cluster:test_node('ejabberd@static-1-12').
      server is reached.
      ok
      (ejabberd@static-1-13)2>  easy_cluster:join_as_master('ejabberd@static-1-12').
      ok
      两个ctrl+c退出ejabberd console.


9. 按装后的初始化工作
-------------------------------

a. 初始化数据库, 自行将localhost替换为域名
    
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
    

10. 功能验证
-----------------


配置详情
===============

目前我们有三台机器，
192.168.1.12
192.168.1.13
192.168.1.15

基础配置如下：
------------------------------------

192.168.1.12  内网主机名: static-1-12
192.168.1.13  内网主机名: static-1-13
192.168.1.15  内网主机名: static-1-15

    1) 把下面四行加入/etc/hosts
    127.0.0.1 cassandra.host
    192.168.1.12 static-1-12
    192.168.1.13 static-1-13
    192.168.1.15 static-1-15
 
    2) 安装JRE，版本：1.7.0_51
    
    3) 安装前提需要的软件包：
    epel repo, nodejs, npm, tar, unzip
    # rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
    # yum install nodejs npm tar unzip

Cassandra配置如下：
--------------------------------------

192.168.1.12  内网主机名: static-1-12
192.168.1.13  内网主机名: static-1-13
192.168.1.15  内网主机名: static-1-15

     1) 添加Datastax Repo文件
     /etc/yum.repos.d/datastax.repo
     2) 安装包：
     jna, dsc20
     # yum install jna dsc20
     3) 修改文件/etc/init.d/cassandra的这一行。
     JAVA_HOME="/usr/java/jre1.7.0_51" 
     4) 修改文件/etc/cassandra/conf/cassandra.yaml
     seeds: 三个内网IP
     listen_address: 本地内网IP
     rpc_address: 本地内网IP
     5）启动cassandra服务，并设置开机启动
     6）三台机器的cassandra和tomcat-usergrid启动后，在192.168.1.12运行命令
     # /usr/bin/curl --user stliu:1234567890 http://localhost:8080/system/database/setup
     # /usr/bin/curl --user stliu:1234567890 http://localhost:8080/system/superuser/setup
     6）检查:
     # /etc/init.d/cassandra status
     /usr/java/jre1.7.0_51
     cassandra (pid  38906) is running...
     # lsof -i :7199
     
Tomcat Usergrid配置如下：
-----------------------------------------

192.168.1.12  内网主机名: static-1-12 
192.168.1.13  内网主机名: static-1-13 
192.168.1.15  内网主机名: static-1-15 

     1）下载[tomcat 7.0.50](http://www.easemob.com/downloads/install/tomcat.tar.gz)压缩包并解压到/opt/tomcat-usergrid
        下载[tomcat_conf.tar.gz](http://www.easemob.com/downloads/install/conf/tomcat_conf.tar.gz)到/tmp目录并解压
     2) 新加系统用户tomcat,无家目录
     3）设置目录/opt/tomcat-usergrid的用户组为tomcat
     4) 拷贝启动脚本从/tmp/conf/usergrid/tomcat-usergrid到/etc/init.d/tomcat-usergrid
     5）拷贝属性文件从/tmp/conf/usergrid/usergrid-custom.properties到/opt/tomcat-usergrid/lib/usergrid-custom.properties
     6）拷贝配置文件从/tmp/conf/usergrid/server.xml到/opt/tomcat-usergrid/conf/server.xml
     7) 删除目录/opt/tomcat-usergrid/webapps下的所有文件，下载[ROOT.war](http://www.easemob.com/downloads/install/ROOT-latest.war)文件到/opt/tomcat-cloudcode/webapps, 重命名为ROOT.war
     8) 启动服务tomcat-usergrid并设置开机启动
     9) 检查:
     # /etc/init.d/tomcat-usergrid status
     Tomcat-usergrid is running with pid: 36902
     # lsof -i :8080

Tomcat cloudcode配置如下：
-----------------------------------------

192.168.1.12  内网主机名: static-1-12
192.168.1.13  内网主机名: static-1-13
192.168.1.15  内网主机名: static-1-15

     1）下载[tomcat 7.0.50](http://www.easemob.com/downloads/install/tomcat.tar.gz)压缩包并解压到/opt/tomcat-cloucode
        下载[tomcat_conf.tar.gz](http://www.easemob.com/downloads/install/conf/tomcat_conf.tar.gz)到/tmp目录并解压
     2) 新加系统用户tomcat,无家目录
     3）设置目录/opt/tomcat-cloudcode的用户组为tomcat
     4) 拷贝启动脚本从/tmp/conf/cloudcode/tomcat-cloudcode到/etc/init.d/tomcat-cloudcode
     5）拷贝配置文件从/tmp/conf/cloudcode/server.xml到/opt/tomcat-cloudcode/conf/server.xml
     6) 删除目录/opt/tomcat-cloudcode/webapps下的所有文件，下载[management-latest.war](http://www.easemob.com/downloads/install/management-latest.war)文件到/opt/tomcat-cloudcode/webapps, 并重命名为management.war 
     7) 启动服务tomcat-cloudcode并设置开机启动
     8) 检查:
     # /etc/init.d/tomcat-cloudcode status
     Tomcat-cloudcode is running with pid: 37259
     # lsof -i :8081

Redis配置如下：
-----------------------------------------

192.168.1.12  内网主机名: static-1-12

    1)安装redis包
    redis, hiredis, hireds-devel
    2)修改配置文件/etc/redis.conf
    bind 192.168.1.12
    3)启动redis服务
    /etc/init.d/redis start
    4)检查
    # /etc/init.d/redis status
    # lsof -i :6379

Pushd配置如下：
-----------------------------------------

192.168.1.12  内网主机名: static-1-12
192.168.1.13  内网主机名: static-1-13
192.168.1.15  内网主机名: static-1-15

     1)下载[Pushd](http://www.easemob.com/downloads/install/pushd-latest.tar.gz)压缩包并解压到/opt/pushd目录
     2)删除目录/opt/pushd/node_modules
     3)在目录/opt/pushd目录下运行命令
     # npm install
     # npm install pm2 -g
     4)拷贝配置文件settings.coffee到目录/opt/pushd
     修改文件中的两行, 192.168.1.12将配置redis服务
     redis_host: '192.168.1.12'
     tcp_port: 7894
     host: 'chat.camito.cn'
     5)在redis服务启动后，启动pm2进程, 并将下面的命令加入/etc/rc.local
     # /usr/bin/pm2 start /opt/pushd/server.js -i max -f
     6)检查：
     # ps aux | grep pm2
     # lsof -i :7894

Nginx, php-fpm 和 upload配置如下：
-----------------------------------------

192.168.1.15  内网主机名: static-1-15

    1)安装nginx包
      pcre, pcre-devel, openssl-devel
      [nginx-1.5.8-1.el6.ngx.x86_64.rpm](http://www.easemob.com/downloads/install/nginx-1.5.8-1.el6.ngx.x86_64.rpm)
    2)删除文件
      /etc/nginx/conf.d/default.conf
      /etc/nginx/conf.d/example_ssl.conf
    3）拷贝文件到
      /etc/nginx/nginx.conf
      /etc/nginx/conf.d/upload.conf
      这个文件里有upload的域名
      server_name upload.camito.cn;
    4) 安装php
       php, php-fpm
    5) 修改文件/etc/php.ini
       cgi.fix_pathinfo=0
    6）拷贝文件到
       /etc/php-fpm.d/www.conf
    7) 启动php-fpm并设置开机启动
    8) 创建目录/opt/web/www, 设置用户组为nginx
    9) 下载[easemob-php.zip](http://www.easemob.com/downloads/install/easemob-php-latest.zip)并解压到/opt/web/www/html, 设置用户组为nginx
    10) 创建目录/opt/web/storage， 设置用户组为nginx
    11) 启动nginx
    12) 检查：
    # /etc/init.d/nginx status
    # lsof -i :9092


负载均衡：
-----------------------------------------

需要给ejabberd的集群配置TCP协议的负载均衡，建议使用LVS，轮询访问三台服务的5222端口.

需要配置nginx代理轮询访问三台服务的8080，8081，7894端口。

     
域名：
-----------------------------------------

需要配置3个域名给nginx服务器代理访问相应的转发端口
* api.camito.cn对应转发端口8080
* cloudcode.camito.cn对应转发端口8081
* pushd.camito.cn对应转发端口7894

需要配置1个域名给LVS的虚拟IP地址
* chat.camito.cn

需要配置1个域名给192.168.1.15的nginx服务器
* upload.camito.cn

