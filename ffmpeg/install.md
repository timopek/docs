FFmpeg 2.2.4
=============

    wget http://www.ffmpeg.org/releases/ffmpeg-2.2.4.tar.gz -P /tmp
    tar zxf ffmepg-2.2.4.tar.gz
    yum install yasm yasm-devel -y

    chown -R easemob.easemob /tmp/ffmpeg-2.2.4
    ./configure --prefix=/home/easemob/apps/opt/ffmpeg --datadir=/home/easemob/apps/data/ffmpeg --logfile=/tmp/ffmpeg-config.log

Check the file in /home/easemob/apps/opt/ffmpeg/ and /home/easemob/apps/data/ffmpeg
