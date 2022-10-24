1. 从es官网下载tar包
https://www.elastic.co/cn/downloads/past-releases/elasticsearch-7-17-6
![download](./img/download.jpg)
上传tar包至3台服务器的/opt目录
2. 配置服务器磁盘  
三台节点执行下列操作
```shell
[root@es-1 ~]# mkdir /data  
[root@es-1 ~]# mkfs.xfs /dev/vdb  
[root@es-1 ~]# mount /dev/vdb /data/     
[root@es-1 ~]# mkdir -p /data/elasticsearch/logs /data/elasticsearch/data
[root@es-1 ~]# echo "/dev/vdb /data                   xfs     defaults        0 0" |tee -a  /etc/fstab
```
3. 优化服务器配置
分别修改三台节点的主机名
```shell
[root@es-1 ~]# hostnamectl set-hostname es-1
[root@es-2 ~]# hostnamectl set-hostname es-2
[root@es-3 ~]# hostnamectl set-hostname es-3
```
三台服务器执行下列操作
```shell
[root@es-1 ~]# vi /etc/hosts
10.17.41.169 es-1
10.17.41.170 es-2
10.17.41.171 es-3

#调整文件句柄
[root@es-1 ~]# vi /etc/security/limits.conf  
*   soft    nofile         165535
*   hard    nofile         165535
*   soft    nproc          165535
*   hard    nproc          165535

#调整可用内存，和连接数复用
[root@es-1 ~]# vi /etc/sysctl.conf
net.ipv4.ip_local_port_range = 1024 65535
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 1
net.ipv4.tcp_max_tw_buckets = 15000
net.core.somaxconn = 65535
vm.max_map_count = 262144

[root@es-1 ~]# sysctl -p
```
4. 安装es

三台服务器执行下列操作
```shell
[root@es-1 ~]# useradd elsearch  #创建elsearch用户
[root@es-1 ~]# passwd elsearch #设置elsearch用户密码
[root@es-1 ~]# cd /opt
[root@es-1 ~]# tar -zxvf /opt/elasticsearch-7.17.6-linux-x86_64.tar.gz
[root@es-1 ~]# mv /opt/elasticsearch-7.17.6 /usr/local/elasticsearch
[root@es-1 ~]# chown -R  elsearch:elsearch /usr/local/elasticsearch
[root@es-1 ~]# chown -R elsearch:elsearch /data/elasticsearch
```
5. 修改配置

es-1节点配置，es-2 ,es-3修改对应地址和节点名称即可
```shell
[root@es-1 ~]# vi /usr/local/elasticsearch/config/elasticsearch.yml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ---------------------------------- Cluster -----------------------------------
#
# Use a descriptive name for your cluster:
#填写集群名称，节点统一
cluster.name: my-es-cluster
#是否可以被选择为master节点
node.master: true
# 是否存储数据
node.data: true
#最大集群节点数，因为3个集群，所以配置3
node.max_local_storage_nodes: 3
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#填写本节点的主机名
node.name: es-1
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ----------------------------------- Paths ------------------------------------
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /data/elasticsearch/data
#
# Path to log files:
#
path.logs: /data/elasticsearch/logs
#
# ----------------------------------- Memory -----------------------------------
#
# Lock the memory on startup:
#
#bootstrap.memory_lock: true
#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ---------------------------------- Network -----------------------------------
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#填写本节点的ip
network.host: 10.17.47.81
#
# Set a custom port for HTTP:
#
http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#填写3个节点的ip
discovery.seed_hosts: ["10.17.47.81", "10.17.47.82", "10.17.47.83"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["es-1", "es-2", "es-3"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# ---------------------------------- Gateway -----------------------------------
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ---------------------------------- Various -----------------------------------
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
#填写各节点证书位置
xpack.security.transport.ssl.keystore.path: certs/es-1/es-1.p12
xpack.security.transport.ssl.truststore.path: certs/es-1/es-1.p12
```  
```shell
[elsearch@es-1 /]$ vi /usr/local/elasticsearch/config/jvm.options
[elsearch@es-2 /]$ vi /usr/local/elasticsearch/config/jvm.options
[elsearch@es-3 /]$ vi /usr/local/elasticsearch/config/jvm.options

#此参数表示es所使用集群最大内存，此参数最大不超过32g，最优设置为服务器内存的百分之70左右
-Xms24g
-Xmx24g


```


6. 配置xpack认证，启用密码
切换至elsearch用户  
```shell
[elsearch@es-1 /]$ su elsearch  
[elsearch@es-2 /]$ su elsearch
[elsearch@es-3 /]$ su elsearch  
```
编写集群证书清单，根据清单生成证书 --pass 设置密码，后续生成keystone需要使用
```shell
[elsearch@es-1 /]$ vi instances.yml 
instances:
    - name: "es-1" 
      ip: 
        - "10.17.47.81"
    - name: "es-2"
      ip:
        - "10.17.47.82"
    - name: "es-3"
      ip:
        - "10.17.47.83"
[elsearch@es-1 /]$ /usr/local/elasticsearch/bin/elasticsearch-certutil cert --silent --in instances.yml --out xpack.zip --pass elsearch
[elsearch@es-1 /]$ unzip xpack.zip 
[elsearch@es-1 /]$ ls
[elsearch@es-1 /]$ rsync -avz es-1 10.17.47.81:/usr/local/elasticsearch/config/certs
[elsearch@es-1 /]$ rsync -avz es-2 10.17.47.82:/usr/local/elasticsearch/config/certs
[elsearch@es-1 /]$ rsync -avz es-3 10.17.47.83:/usr/local/elasticsearch/config/certs
[elsearch@es-1 /]$ /usr/local/elasticsearch/bin/elasticsearch-keystore create
#根据提示填写生成证书时所填写的密码
[elsearch@es-1 /]$ /usr/local/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
[elsearch@es-1 /]$ /usr/local/elasticsearch/bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password

```

7. 启动服务并验证

```shell
[elsearch@es-1 /]$ /usr/local/elasticsearch/bin/elasticsearch -d
[elsearch@es-2 /]$ /usr/local/elasticsearch/bin/elasticsearch -d
[elsearch@es-3 /]$ /usr/local/elasticsearch/bin/elasticsearch -d
#自动生成强密码
[elsearch@es-1 /]$ /usr/local/elasticsearch/bin/elasticsearch-setup-passwords auto
```

访问任意节点地址加9200端口，输入上一个步骤所生成的密码
![Verification](./img/es%E9%AA%8C%E8%AF%81.jpg)

8. 配置监控