1. 从es官网下载tar包
https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-8-0
![download](./img/download.jpg)
上传tar包至3台服务器的/opt目录
2. 配置服务器磁盘  
三台节点执行下列操作
[root@es-1 ~]# mkdir /data  
[root@es-1 ~]# mkfs.xfs /dev/vdb  
[root@es-1 ~]# mount /dev/vdb /data/     
[root@es-1 ~]# mkdir -p /data/elasticsearch/logs /data/elasticsearch/data
[root@es-1 ~]# echo "/dev/vdb /data                   xfs     defaults        0 0" |tee -a  /etc/fstab
3. 优化服务器配置
分别修改三台节点的主机名
[root@es-1 ~]# hostnamectl set-hostname es-1
[root@es-2 ~]# hostnamectl set-hostname es-2
[root@es-3 ~]# hostnamectl set-hostname es-3

三台服务器执行下列操作
[root@es-1 ~]# vi /etc/hosts
10.17.41.169 es-1
10.17.41.170 es-2
10.17.41.171 es-3
[root@es-1 ~]# vi /etc/security/limits.d/90-nproc.conf
elsearch   soft    nproc    4096
[root@es-1 ~]# vi /etc/security/limits.conf 
elsearch   soft   nofile   65536
elsearch   hard   nofile   131072
elsearch   soft   nproc    4096
elsearch   hard   nproc    4096
[root@es-1 ~]# vi /etc/sysctl.conf
vm.max_map_count=655360

4. 安装es

三台服务器执行下列操作
[root@es-1 ~]# useradd elsearch  #创建elsearch用户
[root@es-1 ~]# password elsearch #设置elsearch用户密码
[root@es-1 ~]# cd /opt
[root@es-1 ~]# tar -zxvf /opt/elasticsearch-7.8.0-linux-x86_64.tar.gz
[root@es-1 ~]# mv /opt/elasticsearch-7.8.0 /usr/local/elasticsearch
[root@es-1 ~]# chown -R  elsearch:elsearch /usr/local/elasticsearch
5. 修改配置
6. 开启密码
7. 验证
8. 配置监控