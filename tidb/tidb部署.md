
节点规划  
-------

```shell
#IP 172.16.22.41  
[root@tidb-tiup ~]# hostnamectl set-hostname tidb-tiup  
#IP 172.16.22.1  
[root@tidb-pd1 ~]# hostnamectl set-hostname pd1  
#IP 172.16.22.2  
[root@tidb-pd2 ~]# hostnamectl set-hostname pd2  
#IP 172.16.22.3  
[root@tidb-tikv1 ~]# hostnamectl set-hostname tikv1  
#IP 172.16.22.4  
[root@tidb-tikv1 ~]# hostnamectl set-hostname tikv2  
#IP 172.16.22.5  
[root@tidb-tikv3 ~]# hostnamectl set-hostname tikv3  
#IP 172.16.22.6  
[root@tidb-tiflash1 ~]# hostnamectl set-hostname tiflash1

```

1. 配置服务器磁盘

在7个节点依次执行磁盘初始化命令

```shell
#查看数据盘的设备号
[root@tidb-tiup /]# lsblk 
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0    1T  0 disk 
#使用ext4格式化磁盘，并打上DATA标签，挂载到/data目录
[root@tidb-tiup ~]# mkdir /data
[root@tidb-tiup ~]# mkfs.ext4 -L DATA /dev/sda
[root@tidb-tiup ~]# echo "LABEL=DATA /data ext4 defaults,nodelalloc,noatime 0 2" |tee -a  /etc/fstab
#关闭swap
[root@tidb-tiup ~]# echo "vm.swappiness = 0">> /etc/sysctl.conf
[root@tidb-tiup ~]# swapoff -a && swapon -a
[root@tidb-tiup ~]# sysctl -p
#关闭selinux和防火墙
[root@tidb-tiup ~]# setenforce 0
[root@tidb-tiup ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
[root@tidb-tiup ~]# systemctl disable firewalld.service --now
#时间同步
[root@tidb-tiup ~]# yum -y install chrony
[root@tidb-tiup ~]# systemctl enable --now chronyd
[root@tidb-tiup ~]# chronyc sources

```  

2. 节点优化  

在7个节点同时配置节点优化策略

```shell
[root@tidb-tiup ~]# mkdir /etc/tuned/balanced-tidb-optimal/
[root@tidb-tiup ~]# vi /etc/tuned/balanced-tidb-optimal/tuned.conf
[main]
include=balanced

[cpu]
governor=performance

[vm]
transparent_hugepages=never

[disk]
elevator=noop

[root@tidb-tiup ~]# tuned-adm profile balanced-tidb-optimal
[root@tidb-tiup ~]# echo "fs.file-max = 1000000">> /etc/sysctl.conf
[root@tidb-tiup ~]# echo "net.core.somaxconn = 32768">> /etc/sysctl.conf
[root@tidb-tiup ~]# echo "net.ipv4.tcp_tw_recycle = 0">> /etc/sysctl.conf
[root@tidb-tiup ~]# echo "net.ipv4.tcp_syncookies = 0">> /etc/sysctl.conf
[root@tidb-tiup ~]# echo "vm.overcommit_memory = 1">> /etc/sysctl.conf
[root@tidb-tiup ~]# sysctl -p
[root@tidb-tiup ~]# vi /etc/security/limits.conf
#最后一行增加配置
tidb          soft    nofile          1000000
tidb          hard    nofile          1000000
tidb          soft    stack          32768
tidb           hard    stack          32768
```

3. 配置tidb管理用户

在7台节点执行用户创建  

```shell
[root@tidb-tiup ~]# useradd tidb && \
[root@tidb-tiup ~]# passwd tidb
[root@tidb-tiup ~]# visudo
#在最后一行增加
tidb ALL=(ALL) NOPASSWD: ALL
```

4. 配置tidb-tiup节点远程访问权限

在tidb-tiup节点执行以下命令  

```shell  
[root@tidb-tiup ~]# su tidb
[tidb@tidb-tiup ~]# ssh-keygen -t rsa
[tidb@tidb-tiup ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.22.1
[tidb@tidb-tiup ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.22.2
[tidb@tidb-tiup ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.22.3
[tidb@tidb-tiup ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.22.4
[tidb@tidb-tiup ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.22.5
[tidb@tidb-tiup ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.22.6
[tidb@tidb-tiup ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub 172.16.22.41
[tidb@tidb-tiup ~]# exit
[root@tidb-tiup ~]# vi topology.yaml
```  

```yaml  
global:
  user: "tidb"
  ssh_port: 22
  deploy_dir: "/data/tidb-deploy"
  data_dir: "/data/tidb-data"
server_configs: {}
pd_servers:
- host: 172.16.22.1
- host: 172.16.22.2
tidb_servers:
- host: 172.16.22.1
- host: 172.16.22.2
tikv_servers:
- host: 172.16.22.3
- host: 172.16.22.4
- host: 172.16.22.5
tiflash_servers:
- host: 172.16.22.6
monitoring_servers:
- host: 172.16.22.41
grafana_servers:
- host: 172.16.22.41
alertmanager_servers:
- host: 172.16.22.41
```

5. 配置tidb-tiup节点部署工具并启动集群  

在tidb-tiup节点安装tidb部署工具tiup
```  
[root@tidb-tiup ~]# curl --proto '=https' --tlsv1.2 -sSf https://tiup-mirrors.pingcap.com/install.sh | sh
[root@tidb-tiup ~]# which tiup
[root@tidb-tiup ~]# tiup --binary cluster
```
在tidb-up节点进行节点检测并部署集群
```shell  
#检查集群存在的潜在风险
[root@tidb-tiup ~]# tiup cluster check ./topology.yaml
#自动修复集群存在的潜在风险
[root@tidb-tiup ~]# tiup cluster check ./topology.yaml --apply
[root@tidb-tiup ~]# tiup cluster check ./topology.yaml
#部署 TiDB 集群
[root@tidb-tiup ~]# tiup cluster deploy tidb-g6 v6.1.0 ./topology.yaml
安全启动集群
[root@tidb-tiup ~]# tiup cluster start tidb-g6 --init
预期结果如下，表示启动成功。
Started cluster `tidb-test` successfully.
The root password of TiDB database has been changed.
The new password is: 'y_+3Hwp=*AWz8971s6'.
Copy and record it to somewhere safe, it is only displayed once, and will not be stored.
The generated password can NOT be got again in future.
```

在tidb-tiup节点执行节点查看操作
```shell  
[root@tidb-tiup /]# tiup   cluster  display tidb-g6
tiup is checking updates for component cluster ...
A new version of cluster is available:
   The latest version:         v1.13.1
   Local installed version:    v1.10.3
   Update current component:   tiup update cluster
   Update all components:      tiup update --all

Starting component `cluster`: /root/.tiup/components/cluster/v1.10.3/tiup-cluster display tidb-g6
Cluster type:       tidb
Cluster name:       tidb-g6
Cluster version:    v6.1.0
Deploy user:        tidb
SSH type:           builtin
Dashboard URL:      http://172.16.22.2:2379/dashboard
Grafana URL:        http://172.16.22.41:3000
ID                 Role          Host          Ports                            OS/Arch       Status  Data Dir                           Deploy Dir
--                 ----          ----          -----                            -------       ------  --------                           ----------
172.16.22.41:9093  alertmanager  172.16.22.41  9093/9094                        linux/x86_64  Up      /data/tidb-data/alertmanager-9093  /data/tidb-deploy/alertmanager-9093
172.16.22.41:3000  grafana       172.16.22.41  3000                             linux/x86_64  Up      -                                  /data/tidb-deploy/grafana-3000
172.16.22.1:2379   pd            172.16.22.1   2379/2380                        linux/x86_64  Up|L    /data/tidb-data/pd-2379            /data/tidb-deploy/pd-2379
172.16.22.2:2379   pd            172.16.22.2   2379/2380                        linux/x86_64  Up|UI   /data/tidb-data/pd-2379            /data/tidb-deploy/pd-2379
172.16.22.41:9090  prometheus    172.16.22.41  9090/12020                       linux/x86_64  Up      /data/tidb-data/prometheus-9090    /data/tidb-deploy/prometheus-9090
172.16.22.1:4000   tidb          172.16.22.1   4000/10080                       linux/x86_64  Up      -                                  /data/tidb-deploy/tidb-4000
172.16.22.2:4000   tidb          172.16.22.2   4000/10080                       linux/x86_64  Up      -                                  /data/tidb-deploy/tidb-4000
172.16.22.6:9000   tiflash       172.16.22.6   9000/8123/3930/20170/20292/8234  linux/x86_64  Up      /data/tidb-data/tiflash-9000       /data/tidb-deploy/tiflash-9000
172.16.22.3:20160  tikv          172.16.22.3   20160/20180                      linux/x86_64  Up      /data/tidb-data/tikv-20160         /data/tidb-deploy/tikv-20160
172.16.22.4:20160  tikv          172.16.22.4   20160/20180                      linux/x86_64  Up      /data/tidb-data/tikv-20160         /data/tidb-deploy/tikv-20160
172.16.22.5:20160  tikv          172.16.22.5   20160/20180                      linux/x86_64  Up      /data/tidb-data/tikv-20160         /data/tidb-deploy/tikv-20160
```  