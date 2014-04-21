Keepalived LVS DR
===================

Topology
--------

* LVS master server,
  * Public IP: 10.66.129.139
  * Vitual IP(vip): 10.66.129.149

* Real server,
  * Real Server 1: 10.66.129.140
  * Real Server 2: 10.66.129.117

* System info,
  * OS version: Centos 6.5 x86_64
  * LVS version: ipvsadm-1.26-2.el6.x86_64
  * Keepalived version: keepalived-1.2.7-3.el6.x86_64

LVS master server
-----------------

1. Disable the iptables

    [lvs-master] # iptables -F

2. Enable the IP forwarding
   
    [lvs-master] # vim /etc/sysctl.conf

    net.ipv4.ip_forward = 1
    
    [lvs-master] # sysctl -p

3. Install LVS, keepalived packages
   
    [lvs-master] # yum install keepalived ipvsadm

4. Configue keepalived
Get the file in this path, conf_files/master/etc/keepalived 
   
    [lvs-master] # vim /etc/keepalived/keepalived.conf

5. Start the keepalived service

    [lvs-master] # service keepalived start
    [lvs-master] # chkconfig keepalived on


Real Server
-----------

1. Install, configure and start up the openfire.

 * Install packages,

    [openfire 1] # yum install mysql-server
    [openfire 1] # service mysqld start
    [openfire 1] # wget http://www.igniterealtime.org/downloads/download-landing.jsp?file=openfire/openfire-3.9.1-1.i386.rpm
    [openfire 1] # yum install openfire-3.9.1-1.i386.rpm
    [openfire 1] # yum install mysql-connector-java libldb.i686 libldb-devel.i686

 * Create the mysql openfire and grant the user openfire,

    [openfire 1] # mysql -uroot -p
    mysql> grant all on openfire.* to openfire@'%';
    mysql> set password for openfire@'%'=password("redhat");
    mysql> flush privilegs;
    mysql> Bye
    [openfire 1] # cd /opt/openfire/resources/database
    [openfire 1] # cat openfire_mysql.sql | mysql -uroot openfire;
    
 * Start the service opefire,

    [openfire 1] # service openfire start
    [openfire 1] # chkconfig openfire on

 * Disable the iptables
   
    [openfire 1] # iptables -F
   
2. Configure the real server script, /etc/init.d/lvsrs.
Get the file in this path, conf_files/realserver/etc/init.d/lvsrs
It need to change the value of the VIP in this file.

3. Start the lvs rease server,
    
    [openfire 1] # /etc/init.d/lvsrs start
    [openfire 2] # /etc/init.d/lvsrs start


Check the LVS status
--------------------


    [lvs-master] # ipvsadm -L
    IP Virtual Server version 1.2.1 (size=4096)
    Prot LocalAddress:Port Scheduler Flags
    -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
    TCP  dhcp-129-149.nay.redhat.com: rr persistent 50
    -> dhcp-129-117.nay.redhat.com: Route   1      0          0
    -> dhcp-129-140.nay.redhat.com: Route   2      0          0


Reference:
http://www.cnblogs.com/mchina/archive/2012/05/23/2514728.html
