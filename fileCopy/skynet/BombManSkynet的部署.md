docker run -itd --privileged --name skynet1 -p 3390:22 -p 6000:6000 -v /d/public/DockerShare/Server1:/usr/src/Server centos-skynet /usr/sbin/init

docker run -itd --privileged --name skynet2 -p 3391:22 -p 6001:6000 -v /d/public/DockerShare/Server2:/usr/src/Server centos-skynet /usr/sbin/init

docker run -itd --privileged --name skynet3 -p 3392:22 -p 6002:6000 -v /d/public/DockerShare/Server3:/usr/src/Server centos-skynet /usr/sbin/init :对应的我的


服务器ip地址：10.1.5.66:3392 密码:catserver


docker exec -it skynet1 /bin/bash

docker commit skynet1 centos-skynet

---------------skynet----------------
--环境
yum install -y git
yum -y install autoconf
yum -y install automake
yum -y install make
yum -y install readline-devel
yum -y install gcc

--进入共享目录
cd usr/src/Server1

--下载并安装
git clone https://github.com/cloudwu/skynet.git
cd skynet
make linux

172.17.228.193
255.255.255.240

----------------ssh------------------
yum install openssh-server -y
yum install initscripts -y
service sshd restart

------------密码----------------
yum install passwd -y
密码是 catserver

----------telnet-----------------------
yum install nc -y
ln -s /usr/src/Server/skynet/ /root/skynet

---------常用命令-------------
./skynet bombman/config
./skynet examples/config
3rd/lua/lua bombman/client.lua
3rd/lua/lua examples/client.lua