# 二进制方式部署flannel网络插件

## 前言

在k8s集群中，不同节点中得容器能够互通主要归功于CNI插件，常见得CNI插件有flannel (隧道方式，类似于OSI七层网络模型，通过数据包得再封装得方式，flannel 使用 vxlan 技术为各节点创建一个可以互通的 Pod 网络，使用的端口为 UDP 8472（需要开放该端口，如公有云 AWS 等）。）
calico（路由方案），这里我们学着如何通过二进制得方式去安装flannel网络插件。
flanneld 第一次启动时，从 etcd 获取配置的 Pod 网段信息，为本节点分配一个未使用的地址段，然后创建 `flannedl.1` 网络接口（也可能是其它名称，如flannel1 等）。
flannel 将分配给自己的 Pod 网段信息写入 `/run/flannel/docker` 文件，docker 后续使用这个文件中的环境变量设置 `docker0` 网桥，从而从这个地址段为本节点的所有 Pod 容器分配 IP


## 前提准备
 
 - 安装etcd,开启v2版本的api，且生成相关的授权访问证书
 
 - [下载flannel二进制安装包](https://github.com/coreos/flannel) 这里下载的是0.12版本
 
 - 各个节点docker均已安装好
 
 
 ## 手动往etcd注册网络信息
 
  ```
  #创建名为flannel-networks的文件夹
  curl -k --cert ../ssl/etcd.pem    --key ../ssl/etcd-key.pem https://192.168.0.112:2379/v2/keys/flannel-networks -d dir=true
  
  #注册网络配置信息
  curl -k --cert ../ssl/etcd.pem    --key ../ssl/etcd-key.pem https://192.168.0.112:2379/v2/keys/flannel-networks/config  -XPUT -d value='{"Network": "10.0.0.0/16", "SubnetLen": 24, "SubnetMin": "10.0.1.0","SubnetMax": "10.0.20.0", "Backend": {"Type": "vxlan"}}'
  ```
  [配置选项说明](https://github.com/coreos/flannel/blob/master/Documentation/configuration.md)
  
  ## 编写flanneld.conf 和flanneld.service 服务
  
  vim /opt/k8s/cfg/flanneld.conf
  
  ```
  # flanneld 网络配置前缀
FLANNEL_ETCD_PREFIX="flannel-networks"
# etcd endpoints
ETCD_ENDPOINTS="https://192.168.0.112:2379,https://192.168.0.121:2379,https://192.168.0.149:2379"
# 监听的网卡
FLANNEL_OPTIONS="--iface=enp2s0"
  ```
  
  vim /lib/systemd/system/flanneld.service
  
  ```
  [Unit]
Description=Flanneld overlay address etcd agent
Documentation=https://github.com/coreos/flannel
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
EnvironmentFile=-/opt/k8s/cfg/flanneld.conf 
ExecStart=/opt/k8s/bin/flanneld \
  -etcd-cafile=/opt/etcd/ssl/ca.pem \
  -etcd-certfile=/opt/etcd/ssl/etcd.pem \
  -etcd-keyfile=/opt/etcd/ssl/etcd-key.pem \
  -etcd-endpoints=${ETCD_ENDPOINTS} \
  -etcd-prefix=${FLANNEL_ETCD_PREFIX} \
  -ip-masq
ExecStartPost=/opt/k8s/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=always
RestartSec=5
StartLimitInterval=0

[Install]
WantedBy=multi-user.target
RequiredBy=docker.service

  ```
 
 启动flannel服务
  
  ```
  systemctl daemon-reload
  
  systemctl start flanneld
  ```
  
  ## 修改docker.service文件,增加DOCKER_OPT_BIP 启动项
  
  vim /lib/systemd/system/docker.service
  ```
  [Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
BindsTo=containerd.service
After=network-online.target firewalld.service containerd.service
Wants=network-online.target
Requires=docker.socket

[Service]
Type=notify
EnvironmentFile=-/run/flannel/docker
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock ${DOCKER_OPT_BIP}
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target
  ```
 
 重启docker
 
 ```
 systemctl daemon-reload
 
 systemctl restart docker
 ```
 
  ## 检查是成功
  
  查看网卡信息
  
  ```
  ifconfig


docker0   Link encap:Ethernet  HWaddr 02:42:d2:23:56:c7  
          inet addr:10.0.16.1  Bcast:10.0.16.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

enp2s0    Link encap:Ethernet  HWaddr 50:9a:4c:35:b1:44  
          inet addr:192.168.0.112  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::87e0:5e37:4503:693d/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2130599678 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2252909385 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:239974092187 (239.9 GB)  TX bytes:864015370453 (864.0 GB)

flannel.1 Link encap:Ethernet  HWaddr 1e:6c:48:e6:f1:94  
          inet addr:10.0.16.0  Bcast:0.0.0.0  Mask:255.255.255.255
          UP BROADCAST RUNNING MULTICAST  MTU:1450  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:27 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:672 (672.0 B)  TX bytes:504 (504.0 B)

  ```
  
  不同主机不同网段进行ping测试
  
  ```
  root@k8s-master:/# ping 10.0.13.0
PING 10.0.13.0 (10.0.13.0) 56(84) bytes of data.
64 bytes from 10.0.13.0: icmp_seq=1 ttl=64 time=0.500 ms
64 bytes from 10.0.13.0: icmp_seq=2 ttl=64 time=0.248 ms
64 bytes from 10.0.13.0: icmp_seq=3 ttl=64 time=0.350 ms
64 bytes from 10.0.13.0: icmp_seq=4 ttl=64 time=0.275 ms

  ```
 
