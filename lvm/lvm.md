
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
    
    #create PV
    pvcreate /dev/xvdb1
    #create VG
    vgcreate data /dev/xvdb1
    #Check the totle PE of the VG
    vgdisplay data | grep "Total PE"
    #Create LV with the totle size. 127998 is from the result of the above command
    lvcreate -l 127998 data -n apps
    
给逻辑卷apps创建文件系统ext4
    
    mkfs.ext4 /dev/data/apps 

创建用户easemob
    
    groupadd easemob
    useradd -g easemob -d /home/easemob easemob
    passwd easemob
    mkdir /data/apps -p
    chown -R easemob.easemob /data

在文件/etc/fstab添加挂载点
    
    /dev/data/apps	     /data   ext4	     defaults		   0 0
    
挂载硬盘
    
    mount -a
    ln -s /data/apps /home/easemob/apps


