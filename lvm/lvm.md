
查看硬盘列表
    
    fdisk -l

给xvdb分区

    fdisk -S 56 /dev/xvdb << EOF
    n
    p
    1    
    wq
    EOF

创建逻辑卷apps
    
    pvcreate /dev/xvdb1
    vgcreate data /dev/xvdb1
    vgdisplay data | grep "Total PE"
    lvcreate -l 127998 data -n apps
    
给逻辑卷apps创建文件系统ext4
    
    mkfs.ext4 /dev/data/apps 

创建用户easemob
    
    groupadd easemob
    useradd -g easemob -d /home/easemob easemob
    mkdir apps
    chown -R easemob.easemob /home/easemob/apps
    passwd easemob

在文件/etc/fstab添加挂载点
    
    /dev/data/apps	     /home/easemob/apps   ext4	     defaults		   0 0
    
挂载硬盘
    
    mount -a
    chown -R easemob.easemob /home/easemob/apps


