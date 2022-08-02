cat /etc/security/limits.conf
*   soft    nofile         65535
*   hard    nofile         65535
*   soft    noproc         65535
*   hard    noproc         65535

   26  yum install erlang
   27  systemctl enable rabbitmq-server
   28  systemctl start rabbitmq-server
   29  systemctl status rabbitmq-server
   30  systemctl enable rabbitmq-server
   31  systemctl enable rabbitmq
   32  yum install rabbitmq-server
   33  systemctl enable rabbitmq-server
   34  systemctl start rabbitmq-server
   35  systemctl status rabbitmq-server
   36  Rabbitmq-plugins list
   37  rabbitmq-plugins list
   38  rabbitmq-plugins enable rabbitmq_management
   39  systemctl restart rabbitmq-server.service
   40  cat /var/lib/rabbitmq/.erlang.cookie

[root@rabbitmq-1 rabbitmq]# cat rabbitmq-env.conf 
RABBITMQ_MNESIA_BASE=/data/mnesia
RABBITMQ_LOG_BASE=/data/log
  228  rabbitmqctl  set_vm_memory_high_watermark 0.4
  229  rabbitmqctl  set_vm_memory_high_watermark 0.6
  230  rabbitmqctl set_vm_memory_high_watermark_paging_ratio 0.75
   64  chown -R rabbitmq:rabbitmq  /data/
   88   rabbitmqctl list_policies
   89  rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'


 vi /var/lib/rabbitmq/.erlang.cookie

   43  chmod  777 .erlang.cookie 
   44  vi .erlang.cookie 
   45  chmod 400 .erlang.cookie 
   55  rabbitmqctl stop_app
   56  rabbitmqctl join_cluster rabbit@rabbitmq-1
   57  rabbitmqctl start_app

   92  rabbitmqctl change_cluster_node_type ram
   93  rabbitmqctl start_app
   94  rabbitmqctl  status 
   95  rabbitmqctl  set_vm_memory_high_watermark 0.6
