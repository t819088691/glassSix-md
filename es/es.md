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
[root@es-1 ~]# chown -R elsearch:elsearch /data/elasticsearch
5. 修改配置

es-1节点配置
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
#
cluster.name: my-es-cluster
node.master: true
# 是否存储数据
node.data: true
#最大集群节点数，因为3个集群，所以配置3
node.max_local_storage_nodes: 3
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
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
#
network.host: 10.17.41.169
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
#
discovery.seed_hosts: ["10.17.41.169", "10.17.41.170", "10.17.41.171"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["10.17.41.169", "10.17.41.171"]
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
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.enabled: true
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: /usr/local/elasticsearch/config/certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: /usr/local/elasticsearch/config/certs/elastic-certificates.p12
```  

es-2节点配置
```shell
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
#
cluster.name: my-es-cluster
node.master: true
# 是否存储数据
node.data: true
#最大集群节点数，因为3个集群，所以配置3
node.max_local_storage_nodes: 3
#
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: es-2
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
#
network.host: 10.17.41.170
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
#
discovery.seed_hosts: ["10.17.41.169", "10.17.41.170", "10.17.41.171"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["10.17.41.169", "10.17.41.170"]
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
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.enabled: true
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: /usr/local/elasticsearch/config/certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: /usr/local/elasticsearch/config/certs/elastic-certificates.p12
```
es-3 配置
```shell
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
#
cluster.name: my-es-cluster
node.master: true
# 是否存储数据
node.data: true
#最大集群节点数，因为3个集群，所以配置3
node.max_local_storage_nodes: 3
#
#
# ------------------------------------ Node ------------------------------------
#
# Use a descriptive name for the node:
#
node.name: es-3
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
#
network.host: 10.17.41.171
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
#
discovery.seed_hosts: ["10.17.41.169", "10.17.41.170", "10.17.41.171"]
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
cluster.initial_master_nodes: ["10.17.41.170", "10.17.41.171"]
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
xpack.license.self_generated.type: basic
xpack.security.transport.ssl.enabled: true
http.cors.allow-headers: Authorization,X-Requested-With,Content-Length,Content-Type
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: /usr/local/elasticsearch/config/certs/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: /usr/local/elasticsearch/config/certs/elastic-certificates.p12

```
6. 开启密码
7. 验证
8. 配置监控