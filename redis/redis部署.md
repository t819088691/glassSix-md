1. 从官网下载tar包  
https://download.redis.io/releases/  
三台节点执行下列操作  
wget http://download.redis.io/releases/redis-6.0.6.tar.gz  
2. 配置服务器磁盘  
三台节点执行下列操作  
```shell
[root@redis-1 opt]# mkdir /data  
[root@redis-1 opt]# mkfs.xfs /dev/vdb  
[root@redis-1 opt]# mount /dev/vdb  /data  
[root@redis-1 opt]# vi /etc/fstab  
/dev/vdb /data/                   xfs     defaults        0 0    
[root@redis-1 opt]# mkdir /data/redis-data  
[root@redis-1 opt]# mkdir /data/log  

```
3. 优化系统配置  
三台节点执行下列操作  

```shell
[root@redis-1 opt]# hostnamectl set-hostname redis-1  
[root@redis-1 opt]# yum -y install cpp binutils glibc glibc-kernheaders glibc-common glibc-devel gcc make  
[root@redis-1 opt]# yum -y install centos-release-scl  
[root@redis-1 opt]# yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils  
[root@redis-1 opt]# echo "source /opt/rh/devtoolset-9/enable" >> /etc/profile
[root@redis-1 opt]# source /etc/profile
[root@redis-1 opt]# echo never > /sys/kernel/mm/transparent_hugepage/enabled 
[root@redis-1 opt]# sysctl -w fs.file-max=100000 
```
4. 编译安装redis  
三台节点执行下列操作
```shell
[root@redis-1 opt]# tar -zxvf redis-6.0.6.tar.gz  
[root@redis-1 opt]# cd /opt/redis-6.0.6  
[root@redis-1 opt]# make PREFIX=/usr/local/redis install  
[root@redis-1 opt]# mkdir /usr/local/redis/conf  
[root@redis-1 opt]# cp ./redis.conf /usr/local/redis/conf/  
```
5. 配置启动脚本  
三台节点执行下列操作
```shell
[root@redis-1 opt]# vi /usr/lib/systemd/system/redis.service
[Unit]
Description=redis server daemon
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/redis/bin/redis-server  /usr/local/redis/conf/redis.conf
ExecStop=/bin/kill -s QUIT $MAINPID
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target

[root@redis-1 opt]# systemctl daemon-reload  
[root@redis-1 opt]# systemctl enable --now redis.service  
[root@redis-1 opt]# systemctl status redis  
```
6. 构建主从集群  
修改redis-1的redis.conf配置文件
```shell
bind 0.0.0.0

protected-mode no

port 6379

tcp-backlog 511

timeout 0

tcp-keepalive 300

daemonize yes

supervised no

pidfile /var/run/redis_6379.pid

loglevel notice

logfile /data/log/redis.log

databases 16

always-show-logo yes


save 900 1
save 300 10
save 60 10000

stop-writes-on-bgsave-error yes

rdbcompression yes

rdbchecksum yes

dbfilename dump.rdb

rdb-del-sync-files no

dir /data
#配置数据存放路径

replica-serve-stale-data yes

replica-read-only yes

repl-diskless-sync no

repl-diskless-sync-delay 5

repl-diskless-load disabled

repl-disable-tcp-nodelay no

replica-priority 100

acllog-max-len 128

requirepass ds38n829ef023fsdh
#填写主节点的redis密码

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

lazyfree-lazy-user-del no

appendonly no

appendfilename "appendonly.aof"

appendfsync everysec

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

aof-load-truncated yes

aof-use-rdb-preamble yes

lua-time-limit 5000

slowlog-log-slower-than 10000

slowlog-max-len 128

latency-monitor-threshold 0

notify-keyspace-events ""

hash-max-ziplist-entries 512
hash-max-ziplist-value 64

list-max-ziplist-size -2

list-compress-depth 0

set-max-intset-entries 512

zset-max-ziplist-entries 128
zset-max-ziplist-value 64

hll-sparse-max-bytes 3000

stream-node-max-bytes 4096
stream-node-max-entries 100

activerehashing yes

client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

hz 10

dynamic-hz yes

aof-rewrite-incremental-fsync yes

rdb-save-incremental-fsync yes

jemalloc-bg-thread yes

maxclients 10000

activedefrag yes

active-defrag-ignore-bytes 150mb
 
active-defrag-threshold-lower 10

active-defrag-threshold-upper 100
```
修改redis-2和redis-3的redis.conf配置文件
```shell
bind 0.0.0.0

protected-mode yes

port 6379

tcp-backlog 511

timeout 0

tcp-keepalive 300

daemonize yes

supervised no

loglevel notice

logfile /data/log/redis.log

databases 16

always-show-logo yes

save 900 1
save 300 10
save 60 10000

stop-writes-on-bgsave-error yes

rdbcompression yes

rdbchecksum yes

dbfilename dump.rdb

rdb-del-sync-files no

dir /data
#配置数据存放路径

replicaof 10.17.41.138 6379
#填写主节点的ip 加端口

masterauth ds38n829ef023fsdh
#填写主节点的密码

replica-serve-stale-data yes

replica-read-only yes

repl-diskless-sync no

repl-diskless-sync-delay 5

repl-diskless-load disabled

repl-disable-tcp-nodelay no

replica-priority 100

acllog-max-len 128

requirepass ds38n829ef023fsdh
#从节点redis密码

lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no

lazyfree-lazy-user-del no

appendonly no

appendfilename "appendonly.aof"

appendfsync everysec

no-appendfsync-on-rewrite no

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb

aof-load-truncated yes

aof-use-rdb-preamble yes

lua-time-limit 5000

slowlog-log-slower-than 10000

slowlog-max-len 128

latency-monitor-threshold 0

notify-keyspace-events ""

hash-max-ziplist-entries 512
hash-max-ziplist-value 64

list-max-ziplist-size -2

list-compress-depth 0

set-max-intset-entries 512

zset-max-ziplist-entries 128
zset-max-ziplist-value 64

hll-sparse-max-bytes 3000

stream-node-max-bytes 4096
stream-node-max-entries 100

activerehashing yes

client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

hz 10

dynamic-hz yes

aof-rewrite-incremental-fsync yes

rdb-save-incremental-fsync yes

jemalloc-bg-thread yes

maxclients 10000

activedefrag yes

active-defrag-ignore-bytes 150mb
 
active-defrag-threshold-lower 10

active-defrag-threshold-upper 100
```  
三台节点重启redis服务  
```shell
[root@redis-1 opt]# systemctl restart redis
```
7. 构建哨兵模式(可选)
cp /opt/redis-6.0.6/sentinel.conf  /usr/local/redis/conf/sentinel.conf

修改三处配置
daemonize yes

sentinel monitor mymaster x.x.x.x 6379 2   #地址填写redis主节点地址，密码填写主节点密码

sentinel auth-pass mymaster [masterpassword]

/usr/local/redis/bin/redis-sentinel  /usr/local/redis/conf/sentinel.conf


8. 配置监控