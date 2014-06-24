crontab格式
===========

    
    分 时 日 月 周 命令
    * * * * * command

例子, 每天凌晨12：01运行命令/usr/bin/pm2 restart all

    #crontab -e
    1 0 * * * /usr/bin/pm2 restart all

    #crontab -l


