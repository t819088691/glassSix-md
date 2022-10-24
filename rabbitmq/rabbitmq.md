1. 配置磁盘，配置文件句柄

```shell
#在三台服务器配置磁盘挂载
[root@rabbitmq-1 ~]# mkdir /data  
[root@rabbitmq-1 ~]# mkfs.xfs /dev/vdb  
[root@rabbitmq-1 ~]# mount /dev/vdb /data/     
[root@rabbitmq-1 ~]# mkdir -p /data/log /data/mnesia
[root@rabbitmq-1 ~]# echo "/dev/vdb /data                   xfs     defaults        0 0" |tee -a  /etc/fstab
[root@rabbitmq-1 rabbitmq]# cat /etc/security/limits.conf
*   soft    nofile         65535
*   hard    nofile         65535
*   soft    nproc         65535
*   hard    nproc         65535
```

2. 安装服务并配置

```shell
#在三台节点安装rabbitmq服务
下载erlang和rabbitmq-server的rpm包，并同时上传到3台服务器/opt目录

[root@rabbitmq-1 rabbitmq]# yum install /opt/erlang-23.3.4.6-1.el7.x86_64.rpm
[root@rabbitmq-1 rabbitmq]# yum install rabbitmq-server-3.9.15-1.el7.noarch.rpm
[root@rabbitmq-1 rabbitmq]# chown -R rabbitmq:rabbitmq  /data/

#在三台节点配置数据存储路径，日志存储路径
[root@rabbitmq-1 rabbitmq]# vi /etc/rabbitmq/rabbitmq-env.conf 
RABBITMQ_MNESIA_BASE=/data/mnesia
RABBITMQ_LOG_BASE=/data/log
[root@rabbitmq-1 rabbitmq]# systemctl enable rabbitmq-server
[root@rabbitmq-1 rabbitmq]# systemctl start rabbitmq-server
[root@rabbitmq-1 rabbitmq]# systemctl status rabbitmq-server
[root@rabbitmq-1 rabbitmq]# rabbitmq-plugins list
[root@rabbitmq-1 rabbitmq]# rabbitmq-plugins enable rabbitmq_management
```

3. 优化rabbitmq

```shell
#在三台节点添加LimitNOFILE参数
[root@rabbitmq-1 ~]# vi /usr/lib/systemd/system/rabbitmq-server.service

[Unit]
Description=RabbitMQ broker
After=syslog.target network.target

[Service]
LimitNOFILE=65535
Type=notify
User=rabbitmq
Group=rabbitmq
WorkingDirectory=/var/lib/rabbitmq
ExecStart=/usr/lib/rabbitmq/bin/rabbitmq-server
ExecStop=/usr/lib/rabbitmq/bin/rabbitmqctl stop

[Install]
WantedBy=multi-user.target

#在三台节点从节点优化内核
[root@rabbitmq-1 ~]# vi /etc/sysctl.conf 
fs.file-max=655350


```

4. 构建rabbitmq集群  

```shell

#增加节点内存可用水位
[root@rabbitmq-1 rabbitmq]# rabbitmqctl  set_vm_memory_high_watermark 0.6

#查看主节点的cookie
[root@rabbitmq-1 rabbitmq]# cat /var/lib/rabbitmq/.erlang.cookie
VKSCUUJNSTEBMSJADHPL

#将cookie写入两台从节点，并修改权限为400
[root@rabbitmq-2 rabbitmq]# chmod  777 /var/lib/rabbitmq/.erlang.cookie 
[root@rabbitmq-2 rabbitmq]# vi .erlang.cookie 
[root@rabbitmq-2 rabbitmq]# chmod 400 .erlang.cookie 
[root@rabbitmq-2 rabbitmq]# rabbitmqctl stop_app

#将两台从节点加入集群
[root@rabbitmq-2 rabbitmq]# rabbitmqctl join_cluster rabbit@rabbitmq-1
[root@rabbitmq-2 rabbitmq]# rabbitmqctl start_app

#将两台从节点修改节点类型为内存节点
[root@rabbitmq-2 rabbitmq]# rabbitmqctl change_cluster_node_type ram

#将两台从节点增加节点内存可用水位
[root@rabbitmq-2 rabbitmq]# rabbitmqctl  set_vm_memory_high_watermark 0.6

#在主节点配置用户登录rabbitmq web
[root@rabbitmq-1 rabbitmq]# rabbitmqctl add_user 【用户名】 【密码】
[root@rabbitmq-1 rabbitmq]# rabbitmqctl set_user_tags 【用户名】 administrator
[root@rabbitmq-1 rabbitmq]# rabbitmqctl set_permissions -p / 【用户名】 ".*" ".*" ".*"

#在主节点增加策略
[root@rabbitmq-1 rabbitmq]# rabbitmqctl list_policies
[root@rabbitmq-1 rabbitmq]# rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'
```

5. 登录验证
