Install newrelic
===============

    rpm -Uvh https://yum.newrelic.com/pub/newrelic/el5/x86_64/newrelic-repo-5-3.noarch.rpm
    yum install newrelic-sysmond
    nrsysmond-config --set license_key=a2332db36688545714936cbd7ab4baacc10672b1
    /etc/init.d/newrelic-sysmond start


Install ali monitor script
===========================
   
    wget http://update.aegis.aliyun.com/download/quartz_install.sh
    sh quartz_install.sh
