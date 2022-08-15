1. 安装部署机
在部署机安装ansible，kubekey工具

```shell
[root@dlj-ecs-prometheus /]# yum -y install ansible
[root@dlj-ecs-prometheus /]# ansible --version
ansible 2.9.27
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr 11 2018, 07:36:10) [GCC 4.8.5 20150623 (Red Hat 4.8.5-28)]

上传 kubekey-v2.2.1-linux-amd64.tar 压缩包到部署机
[root@dlj-ecs-prometheus opt]# tar -zxvf kubekey-v2.2.1-linux-amd64.tar.gz
kk
[root@dlj-ecs-prometheus opt]# ./kk  version
version.BuildInfo{Version:"2.2.1", GitCommit:"c056977c", GitTreeState:"", GoVersion:"go1.17.11"}
```

2. 编辑配置文件，批量编辑k8s服务器

* 编辑k8s-ansible文件夹中的host.init文件  
* 在all填写所有节点的主机名和ip  
* 在master填写master节点主机名  
* 在worker填写worker节点主机名  
* 在all:vars 填写所有节点的ssh端口，填写所有节点的用户名和密码

```shell
[all]
k8s-master1 ansible_host=10.17.46.31 ip=10.17.46.31
k8s-master2 ansible_host=10.17.46.32 ip=10.17.46.32
k8s-master3 ansible_host=10.17.46.33 ip=10.17.46.33
k8s-worker1 ansible_host=10.17.46.34 ip=10.17.46.34
k8s-worker2 ansible_host=10.17.46.35 ip=10.17.46.35
k8s-worker3 ansible_host=10.17.46.36 ip=10.17.46.36
k8s-worker4 ansible_host=10.17.46.37 ip=10.17.46.37
k8s-worker5 ansible_host=10.17.46.38 ip=10.17.46.38
k8s-worker6 ansible_host=10.17.46.39 ip=10.17.46.39
k8s-worker7 ansible_host=10.17.46.40 ip=10.17.46.40
k8s-worker8 ansible_host=10.17.46.41 ip=10.17.46.41
k8s-worker9 ansible_host=10.17.46.42 ip=10.17.46.42
k8s-worker10 ansible_host=10.17.46.43 ip=10.17.46.43
k8s-worker11 ansible_host=10.17.46.44 ip=10.17.46.44
k8s-worker12 ansible_host=10.17.46.45 ip=10.17.46.45
k8s-worker13 ansible_host=10.17.46.46 ip=10.17.46.46
k8s-worker14 ansible_host=10.17.46.47 ip=10.17.46.47
k8s-worker15 ansible_host=10.17.46.48 ip=10.17.46.48
[all:vars]
ansible_ssh_user=root 
ansible_ssh_port=22 
ansible_ssh_pass="SzsbngChj@1234"

[master]
k8s-master1
k8s-master2
k8s-master3

[worker]
k8s-worker1
k8s-worker2
k8s-worker3
k8s-worker4
k8s-worker5
k8s-worker6
k8s-worker7
k8s-worker8
k8s-worker9
k8s-worker10
k8s-worker11
k8s-worker12
k8s-worker13
k8s-worker14
k8s-worker15


[newnode]

[k8s:children]
master
worker
#newnode
```
3. 初始化k8s服务器  
* 上传编辑修改后的ansible剧本k8s-ansible.zip  
* 使用unzip解压然后cd至k8s-ansible 目录  
```shell
[root@dlj-ecs-prometheus opt]# unzip k8s-ansible.zip 
Archive:  k8s-ansible.zip
   creating: k8s-ansible/
  inflating: k8s-ansible/config-sample.yaml  
   creating: k8s-ansible/group_vars/
 extracting: k8s-ansible/group_vars/all  
  inflating: k8s-ansible/hosts.ini  
  inflating: k8s-ansible/init-master.yaml  
  inflating: k8s-ansible/init-worker.yaml  
   creating: k8s-ansible/roles/
   creating: k8s-ansible/roles/1_yum/
   creating: k8s-ansible/roles/1_yum/files/
  inflating: k8s-ansible/roles/1_yum/files/CentOS-Base.repo  
  inflating: k8s-ansible/roles/1_yum/files/docker.repo  
  inflating: k8s-ansible/roles/1_yum/files/kubernetes.repo  
   creating: k8s-ansible/roles/1_yum/tasks/
  inflating: k8s-ansible/roles/1_yum/tasks/main.yaml  
   creating: k8s-ansible/roles/2_update_kernal/
   creating: k8s-ansible/roles/2_update_kernal/files/
  inflating: k8s-ansible/roles/2_update_kernal/files/updatekernal.sh  
   creating: k8s-ansible/roles/2_update_kernal/tasks/
  inflating: k8s-ansible/roles/2_update_kernal/tasks/main.yaml  
   creating: k8s-ansible/roles/3_host_set/
   creating: k8s-ansible/roles/3_host_set/tasks/
  inflating: k8s-ansible/roles/3_host_set/tasks/main.yaml  
   creating: k8s-ansible/roles/3_host_set/templates/
  inflating: k8s-ansible/roles/3_host_set/templates/hosts.j2  
   creating: k8s-ansible/roles/4_init/
   creating: k8s-ansible/roles/4_init/tasks/
  inflating: k8s-ansible/roles/4_init/tasks/main.yaml  
   creating: k8s-ansible/roles/5_master_install_package/
   creating: k8s-ansible/roles/5_master_install_package/tasks/
  inflating: k8s-ansible/roles/5_master_install_package/tasks/main.yaml  
   creating: k8s-ansible/roles/5_node_install_package/
   creating: k8s-ansible/roles/5_node_install_package/tasks/
  inflating: k8s-ansible/roles/5_node_install_package/tasks/main.yaml  
   creating: k8s-ansible/roles/6_sysconf/
   creating: k8s-ansible/roles/6_sysconf/files/
  inflating: k8s-ansible/roles/6_sysconf/files/ipvs.modules  
  inflating: k8s-ansible/roles/6_sysconf/files/k8s.conf  
   creating: k8s-ansible/roles/6_sysconf/tasks/
  inflating: k8s-ansible/roles/6_sysconf/tasks/main.yaml  
   creating: k8s-ansible/roles/7_conf_containerd/
   creating: k8s-ansible/roles/7_conf_containerd/files/
  inflating: k8s-ansible/roles/7_conf_containerd/files/config.toml  
   creating: k8s-ansible/roles/7_conf_containerd/handlers/
  inflating: k8s-ansible/roles/7_conf_containerd/handlers/main.yaml  
   creating: k8s-ansible/roles/7_conf_containerd/tasks/
  inflating: k8s-ansible/roles/7_conf_containerd/tasks/main.yaml  
   creating: k8s-ansible/roles/8_config_disk/
   creating: k8s-ansible/roles/8_config_disk/files/
  inflating: k8s-ansible/roles/8_config_disk/files/config_disk.sh  
   creating: k8s-ansible/roles/8_config_disk/tasks/
  inflating: k8s-ansible/roles/8_config_disk/tasks/main.yaml 
[root@dlj-ecs-prometheus opt]# cd k8s-ansible
[root@dlj-ecs-prometheus k8s-ansible]# ls
config-sample.yaml  group_vars  hosts.ini  init-master.yaml  init-worker.yaml  roles
```
测试部署机与集群的连通性
```shell
[root@dlj-ecs-prometheus k8s-ansible]# ansible -i hosts.ini all -m shell -a "hostname"
k8s-master3 | CHANGED | rc=0 >>
dljb-k8s-master-3.novalocal
k8s-master2 | CHANGED | rc=0 >>
dljb-k8s-master-2.novalocal
k8s-master1 | CHANGED | rc=0 >>
dljb-k8s-master-1.novalocal
k8s-worker2 | CHANGED | rc=0 >>
dljb-k8s-worker-2.novalocal
k8s-worker1 | CHANGED | rc=0 >>
dljb-k8s-worker-1.novalocal
k8s-worker3 | CHANGED | rc=0 >>
dljb-k8s-worker-3.novalocal
k8s-worker4 | CHANGED | rc=0 >>
dljb-k8s-worker-4.novalocal
k8s-worker5 | CHANGED | rc=0 >>
dljb-k8s-worker-5.novalocal
k8s-worker6 | CHANGED | rc=0 >>
dljb-k8s-worker-6.novalocal
k8s-worker7 | CHANGED | rc=0 >>
dljb-k8s-worker-7.novalocal
k8s-worker8 | CHANGED | rc=0 >>
dljb-k8s-worker-8.novalocal
k8s-worker10 | CHANGED | rc=0 >>
dljb-k8s-worker-10.novalocal
k8s-worker12 | CHANGED | rc=0 >>
dljb-k8s-worker-12.novalocal
k8s-worker11 | CHANGED | rc=0 >>
dljb-k8s-worker-11.novalocal
k8s-worker9 | CHANGED | rc=0 >>
dljb-k8s-worker-9.novalocal
k8s-worker15 | CHANGED | rc=0 >>
dljb-k8s-worker-15.novalocal
k8s-worker14 | CHANGED | rc=0 >>
dljb-k8s-worker-14.novalocal
k8s-worker13 | CHANGED | rc=0 >>
dljb-k8s-worker-13.novalocal
```
初始化master节点
```shell
[root@dlj-ecs-prometheus k8s-ansible]# ansible-playbook -i hosts.ini  init-master.yaml

PLAY RECAP *******************************************************************************************************************************************************************************************************
k8s-master1                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-master2                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-master3                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```
初始化worker节点
```shell
[root@dlj-ecs-prometheus k8s-ansible]# ansible-playbook -i hosts.ini  init-worker.yaml
PLAY RECAP *******************************************************************************************************************************************************************************************************
k8s-worker1                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker10               : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker11               : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker12               : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker13               : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker14               : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker15               : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker2                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker3                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker4                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker5                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker6                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker7                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker8                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
k8s-worker9                : ok=6    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

```
4. 编辑集群配置文件

* 在hosts中填写所有节点的ip和用户名密码  
* spec.roleGroups.etcd 和spec.roleGroups.control-plane 填写3台master的name  
* 在sepc.etcd中填写3台master节点的ip用逗号加空格分割  
* 取消 internalLoadbalancer: haproxy 的注释  
```yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: sample
spec:
  hosts:
  - {name: k8s-master1, address: 10.17.46.31, internalAddress: 10.17.46.31, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-master2, address: 10.17.46.32, internalAddress: 10.17.46.32, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-master3, address: 10.17.46.33, internalAddress: 10.17.46.33, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker1, address: 10.17.46.34, internalAddress: 10.17.46.34, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker2, address: 10.17.46.35, internalAddress: 10.17.46.35, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker3, address: 10.17.46.36, internalAddress: 10.17.46.36, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker4, address: 10.17.46.37, internalAddress: 10.17.46.37, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker5, address: 10.17.46.38, internalAddress: 10.17.46.38, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker6, address: 10.17.46.39, internalAddress: 10.17.46.39, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker7, address: 10.17.46.40, internalAddress: 10.17.46.40, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker8, address: 10.17.46.41, internalAddress: 10.17.46.41, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker9, address: 10.17.46.42, internalAddress: 10.17.46.42, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker10, address: 10.17.46.43, internalAddress: 10.17.46.43, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker11, address: 10.17.46.44, internalAddress: 10.17.46.44, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker12, address: 10.17.46.45, internalAddress: 10.17.46.45, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker13, address: 10.17.46.46, internalAddress: 10.17.46.46, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker14, address: 10.17.46.47, internalAddress: 10.17.46.47, user: root, password: "SzsbngChj@1234"}
  - {name: k8s-worker15, address: 10.17.46.48, internalAddress: 10.17.46.48, user: root, password: "SzsbngChj@1234"}
  roleGroups:
    etcd:
    - k8s-master1
    - k8s-master2
    - k8s-master3
    control-plane: 
    - k8s-master1
    - k8s-master2
    - k8s-master3
    worker:
    - k8s-worker1
    - k8s-worker2
    - k8s-worker3
    - k8s-worker4
    - k8s-worker5
    - k8s-worker6
    - k8s-worker7
    - k8s-worker8
    - k8s-worker9
    - k8s-worker10
    - k8s-worker11
    - k8s-worker12
    - k8s-worker13
    - k8s-worker14
    - k8s-worker15
  controlPlaneEndpoint:
    ## Internal loadbalancer for apiservers 
    internalLoadbalancer: haproxy

    domain: lb.kubesphere.local
    address: ""
    port: 6443
  kubernetes:
    version: v1.22.10
    clusterName: cluster.local
    autoRenewCerts: true
    containerManager: containerd
  etcd:
    type: kubekey
  network:
    plugin: calico
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
    ## multus support. https://github.com/k8snetworkplumbingwg/multus-cni
    multusCNI:
      enabled: false
  registry:
    privateRegistry: ""
    namespaceOverride: ""
    registryMirrors: []
    insecureRegistries: []
  addons: []



---
apiVersion: installer.kubesphere.io/v1alpha1
kind: ClusterConfiguration
metadata:
  name: ks-installer
  namespace: kubesphere-system
  labels:
    version: v3.3.0
spec:
  persistence:
    storageClass: ""
  authentication:
    jwtSecret: ""
  zone: ""
  local_registry: ""
  namespace_override: ""
  # dev_tag: ""
  etcd:
    monitoring: true
    endpointIps: 10.17.46.31, 10.17.46.32, 10.17.46.33
    port: 2379
    tlsEnable: true
  common:
    core:
      console:
        enableMultiLogin: true
        port: 30880
        type: NodePort
    # apiserver:
    #  resources: {}
    # controllerManager:
    #  resources: {}
    redis:
      enabled: false
      volumeSize: 2Gi
    openldap:
      enabled: false
      volumeSize: 2Gi
    minio:
      volumeSize: 20Gi
    monitoring:
      # type: external
      endpoint: http://prometheus-operated.kubesphere-monitoring-system.svc:9090
      GPUMonitoring:
        enabled: false
    gpu:
      kinds:
      - resourceName: "nvidia.com/gpu"
        resourceType: "GPU"
        default: true
    es:
      # master:
      #   volumeSize: 4Gi
      #   replicas: 1
      #   resources: {}
      # data:
      #   volumeSize: 20Gi
      #   replicas: 1
      #   resources: {}
      logMaxAge: 7
      elkPrefix: logstash
      basicAuth:
        enabled: false
        username: ""
        password: ""
      externalElasticsearchHost: ""
      externalElasticsearchPort: ""
  alerting:
    enabled: false
    # thanosruler:
    #   replicas: 1
    #   resources: {}
  auditing:
    enabled: false
    # operator:
    #   resources: {}
    # webhook:
    #   resources: {}
  devops:
    enabled: false
    # resources: {}
    jenkinsMemoryLim: 2Gi
    jenkinsMemoryReq: 1500Mi
    jenkinsVolumeSize: 8Gi
    jenkinsJavaOpts_Xms: 1200m
    jenkinsJavaOpts_Xmx: 1600m
    jenkinsJavaOpts_MaxRAM: 2g
  events:
    enabled: false
    # operator:
    #   resources: {}
    # exporter:
    #   resources: {}
    # ruler:
    #   enabled: true
    #   replicas: 2
    #   resources: {}
  logging:
    enabled: false
    logsidecar:
      enabled: true
      replicas: 2
      # resources: {}
  metrics_server:
    enabled: true
  monitoring:
    storageClass: ""
    node_exporter:
      port: 9100
      # resources: {}
    # kube_rbac_proxy:
    #   resources: {}
    # kube_state_metrics:
    #   resources: {}
    # prometheus:
    #   replicas: 1
    #   volumeSize: 20Gi
    #   resources: {}
    #   operator:
    #     resources: {}
    # alertmanager:
    #   replicas: 1
    #   resources: {}
    # notification_manager:
    #   resources: {}
    #   operator:
    #     resources: {}
    #   proxy:
    #     resources: {}
    gpu:
      nvidia_dcgm_exporter:
        enabled: false
        # resources: {}
  multicluster:
    clusterRole: none
  network:
    networkpolicy:
      enabled: false
    ippool:
      type: none
    topology:
      type: none
  openpitrix:
    store:
      enabled: false
  servicemesh:
    enabled: false
    istio:
      components:
        ingressGateways:
        - name: istio-ingressgateway
          enabled: false
        cni:
          enabled: false
  edgeruntime:
    enabled: false
    kubeedge:
      enabled: false
      cloudCore:
        cloudHub:
          advertiseAddress:
            - ""
        service:
          cloudhubNodePort: "30000"
          cloudhubQuicNodePort: "30001"
          cloudhubHttpsNodePort: "30002"
          cloudstreamNodePort: "30003"
          tunnelNodePort: "30004"
        # resources: {}
        # hostNetWork: false
      iptables-manager:
        enabled: true
        mode: "external"
        # resources: {}
      # edgeService:
      #   resources: {}
  terminal:
    timeout: 600
```
5. 使用kk工具部署集群
```shell
[root@dlj-ecs-prometheus opt]# ./kk  create cluster -f C环境-k8s-ansible/config-sample.yaml 


 _   __      _          _   __           
| | / /     | |        | | / /           
| |/ / _   _| |__   ___| |/ /  ___ _   _ 
|    \| | | | '_ \ / _ \    \ / _ \ | | |
| |\  \ |_| | |_) |  __/ |\  \  __/ |_| |
\_| \_/\__,_|_.__/ \___\_| \_/\___|\__, |
                                    __/ |
                                   |___/

10:30:53 CST [GreetingsModule] Greetings
10:30:53 CST message: [k8s-worker15]
Greetings, KubeKey!
10:30:53 CST message: [k8s-master3]
Greetings, KubeKey!
10:30:53 CST message: [k8s-master2]
Greetings, KubeKey!
10:30:53 CST message: [k8s-worker4]
Greetings, KubeKey!
10:30:53 CST message: [k8s-worker3]
Greetings, KubeKey!
10:30:53 CST message: [k8s-worker12]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker10]
Greetings, KubeKey!
10:30:54 CST message: [k8s-master1]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker2]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker9]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker11]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker13]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker6]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker5]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker7]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker8]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker14]
Greetings, KubeKey!
10:30:54 CST message: [k8s-worker1]
Greetings, KubeKey!
10:30:54 CST success: [k8s-worker15]
10:30:54 CST success: [k8s-master3]
10:30:54 CST success: [k8s-master2]
10:30:54 CST success: [k8s-worker4]
10:30:54 CST success: [k8s-worker3]
10:30:54 CST success: [k8s-worker12]
10:30:54 CST success: [k8s-worker10]
10:30:54 CST success: [k8s-master1]
10:30:54 CST success: [k8s-worker2]
10:30:54 CST success: [k8s-worker9]
10:30:54 CST success: [k8s-worker11]
10:30:54 CST success: [k8s-worker13]
10:30:54 CST success: [k8s-worker6]
10:30:54 CST success: [k8s-worker5]
10:30:54 CST success: [k8s-worker7]
10:30:54 CST success: [k8s-worker8]
10:30:54 CST success: [k8s-worker14]
10:30:54 CST success: [k8s-worker1]
10:30:54 CST [NodePreCheckModule] A pre-check on nodes
10:30:55 CST success: [k8s-worker1]
10:30:55 CST success: [k8s-worker5]
10:30:55 CST success: [k8s-worker4]
10:30:55 CST success: [k8s-worker10]
10:30:55 CST success: [k8s-worker15]
10:30:55 CST success: [k8s-master3]
10:30:55 CST success: [k8s-worker6]
10:30:55 CST success: [k8s-worker9]
10:30:55 CST success: [k8s-worker7]
10:30:55 CST success: [k8s-master1]
10:30:55 CST success: [k8s-worker11]
10:30:55 CST success: [k8s-worker13]
10:30:55 CST success: [k8s-worker12]
10:30:55 CST success: [k8s-worker14]
10:30:55 CST success: [k8s-worker8]
10:30:55 CST success: [k8s-worker2]
10:30:55 CST success: [k8s-worker3]
10:30:55 CST success: [k8s-master2]
10:30:55 CST [ConfirmModule] Display confirmation form
+--------------+------+------+---------+----------+-------+-------+---------+-----------+--------+--------+------------+------------+-------------+------------------+--------------+
| name         | sudo | curl | openssl | ebtables | socat | ipset | ipvsadm | conntrack | chrony | docker | containerd | nfs client | ceph client | glusterfs client | time         |
+--------------+------+------+---------+----------+-------+-------+---------+-----------+--------+--------+------------+------------+-------------+------------------+--------------+
| k8s-master1  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-master2  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-master3  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker1  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker2  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker3  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker4  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker5  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker6  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker7  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker8  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker9  | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker10 | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker11 | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker12 | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker13 | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker14 | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
| k8s-worker15 | y    | y    | y       | y        | y     | y     | y       | y         | y      |        |            | y          |             |                  | CST 10:30:55 |
+--------------+------+------+---------+----------+-------+-------+---------+-----------+--------+--------+------------+------------+-------------+------------------+--------------+

This is a simple check of your environment.
Before installation, ensure that your machines meet all requirements specified at
https://github.com/kubesphere/kubekey#requirements-and-recommendations

Continue this installation? [yes/no]: yes
```
等待集群部署完成  
```shell
10:31:34 CST success: [LocalHost]
10:31:34 CST [NodeBinariesModule] Download installation binaries
10:31:34 CST message: [localhost]
downloading amd64 kubeadm v1.22.10 ...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 43.7M  100 43.7M    0     0  2916k      0  0:00:15  0:00:15 --:--:-- 4361k
```
6. 验证登录

7. 开启相关插件
