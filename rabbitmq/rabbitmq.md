[root@rabbitmq-1 rabbitmq]# cat /etc/security/limits.conf
*   soft    nofile         65535
*   hard    nofile         65535
*   soft    noproc         65535
*   hard    noproc         65535

yum install erlang
systemctl enable rabbitmq-server
systemctl start rabbitmq-server
systemctl status rabbitmq-server
systemctl enable rabbitmq-server
systemctl enable rabbitmq
yum install rabbitmq-server
systemctl enable rabbitmq-server
systemctl start rabbitmq-server
systemctl status rabbitmq-server
Rabbitmq-plugins list
rabbitmq-plugins list
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server.service
cat /var/lib/rabbitmq/.erlang.cookie

[root@rabbitmq-1 rabbitmq]# cat rabbitmq-env.conf 
RABBITMQ_MNESIA_BASE=/data/mnesia
RABBITMQ_LOG_BASE=/data/log
rabbitmqctl  set_vm_memory_high_watermark 0.6
chown -R rabbitmq:rabbitmq  /data/
rabbitmqctl list_policies
rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'


vi /var/lib/rabbitmq/.erlang.cookie

chmod  777 .erlang.cookie 
vi .erlang.cookie 
chmod 400 .erlang.cookie 
rabbitmqctl stop_app
rabbitmqctl join_cluster rabbit@rabbitmq-1
rabbitmqctl start_app
rabbitmqctl change_cluster_node_type ram
rabbitmqctl start_app
rabbitmqctl  status 
rabbitmqctl  set_vm_memory_high_watermark 0.6
