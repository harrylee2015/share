# 部署带本地自签CA认证的etcd集群

## 前言

  etcd在生产环境中部署时往往都是采用https方式进行访问，这往往需要我们本地搞个自签证书，下面我们一起来学习一下如何去生成和部署


## 下载所需的包(cfssl,生成证书工具)

```
mkdir /opt/etcd/{cfg,ssl,bin} -p
cd /opt/etcd/
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64 
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl*

#拷贝命令
cp -v cfssl_linux-amd64 /usr/local/bin/cfssl
cp -v cfssljson_linux-amd64 /usr/local/bin/cfssljson
cp -v cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
ls /usr/local/bin/cfssl*

```

## 使用CFSSL创建CA证书以及etcd的TLS认证证书

```
cd /opt/etcd/ssl

#创建 CA 配置文件（ca-config.json）
vim ca-config.json
{
  "signing": {
    "default": {
      "expiry": "876000h"
    },
    "profiles": {
      "etcd": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "876000h"
      }
    }
  }
}

"字段说明"
"ca-config.json"：可以定义多个profiles，分别指定不同的过期时间、使用场景等参数,这里证书有效期是100年；后续在签名证书时使用某个 profile；
"signing"：表示该证书可用于签名其它证书；生成的 ca.pem 证书中 CA=TRUE；
"server auth"：表示client可以用该 CA 对server提供的证书进行验证；
"client auth"：表示server可以用该CA对client提供的证书进行验证；

#创建 CA 证书签名请求（ca-csr.json）
vim ca-csr.json
{
  "CN": "etcd",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "nanjing",
      "L": "nanjing",
      "O": "k8s-etcd-cluster",
      "OU": "System"
    }
  ]
}

"CN"：Common Name，etcd 从证书中提取该字段作为请求的用户名 (User Name)；浏览器使用该字段验证网站是否合法；
"O"：Organization，etcd 从证书中提取该字段作为请求用户所属的组 (Group)；
这两个参数在后面的kubernetes启用RBAC模式中很重要，因为需要设置kubelet、admin等角色权限，那么在配置证书的时候就必须配置对了，具体后面在部署kubernetes的时候会进行讲解。
"在etcd这两个参数没太大的重要意义，跟着配置就好。"

#生成 CA 证书和私钥
cfssl gencert -initca ca-csr.json | cfssljson -bare ca


#创建etcd的TLS认证证书
#创建 etcd证书签名请求（etcd-csr.json）
vim etcd-csr.json
{
  "CN": "etcd",
  "hosts": [
    "127.0.0.1",
    "192.168.0.112",
    "192.168.0.121",
    "192.168.0.149"

  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "nanjing",
      "L": "nanjing",
      "O": "k8s-etcd-cluster",
      "OU": "System"
    }
  ]
}

如果 hosts 字段不为空则需要指定授权使用该证书的 IP 或域名列表，由于该证书后续被 etcd 集群使用，所以填写IP即可。
       
#生成 etcd证书和私钥 
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd-csr.json | cfssljson -bare etcd

#将TLS 认证文件拷贝至证书目录下
 
 就在本目录下生成得，所以不需要操作

```

## 安装etcd

 以下是每个节点都要做

 ```
   # 到etcd项目下下载安装包
  wget xxxx
  
  tar -xvf  xxx.tar.gz
  
  #将二进制文件移动到bin目录下
  
  cp etcd etcdctl /opt/etcd/bin
  
 ```
 
 ## 将之前生成得证书拷贝到 ssl目录下
 
   略
   
## 编写 etcd.conf 配置文件   
 
  vim /opt/etcd/cfg/etcd.conf
  
 ```
 #[member]
#etcd实例名称
NAME="etcd1"
#etcd数据保存目录，这个目录需要提前创好
DATA_DIR="/var/lib/etcd"
#集群内部通信使用得URL
LISTEN_PEER_URLS="https://192.168.0.112:2380"
#供外部客户端使用得URL
LISTEN_CLIENT_URLS="https://192.168.0.112:2379,https://127.0.0.1:2379"
#广播给外部客户端使用的url
ADVERTISE_CLIENT_URLS="https://192.168.0.112:2379,https://127.0.0.1:2379"

#[cluster]
#广播给集群内其他成员访问得URL
INITIAL_ADVERTISE_PEER_URLS="https://192.168.0.112:2380"
#集群名称
INITIAL_CLUSTER_TOKEN="k8s-etcd-cluster"
#初始集群状态，new为新建集群
INITIAL_CLUSTER_STATE="new"
##初始集群成员列表
INITIAL_CLUSTER="etcd1=https://192.168.0.112:2380,etcd2=https://192.168.0.121:2380,etcd3=https://192.168.0.149:2380"

 ```
 
 ## 编写systemd etcd.service服务文件
  
   系统不同，路径可能不一样，这里时ubutnu 16.04
  ```
  vim /lib/systemd/system/etcd.service
 ```
 
 ```
 [Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/opt/etcd/cfg/etcd.conf
# set GOMAXPROCS to number of processors
ExecStart=/opt/etcd/bin/etcd --name ${NAME} \
  --cert-file=/opt/etcd/ssl/etcd.pem \
  --key-file=/opt/etcd/ssl/etcd-key.pem \
  --peer-cert-file=/opt/etcd/ssl/etcd.pem \
  --peer-key-file=/opt/etcd/ssl/etcd-key.pem \
  --trusted-ca-file=/opt/etcd/ssl/ca.pem \
  --peer-trusted-ca-file=/opt/etcd/ssl/ca.pem \
  --advertise-client-urls ${ADVERTISE_CLIENT_URLS} \
  --listen-peer-urls ${LISTEN_PEER_URLS} \
  --listen-client-urls ${LISTEN_CLIENT_URLS} \
  --initial-advertise-peer-urls ${INITIAL_ADVERTISE_PEER_URLS} \
  --initial-cluster-token ${INITIAL_CLUSTER_TOKEN} \
  --initial-cluster ${INITIAL_CLUSTER} \
  --initial-cluster-state new \
  --data-dir=${DATA_DIR}

Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
  
  ```
  
  ## 启动服务
  
  ```
  # 重载配置
  systemctl daemon-reload
  
  #启动
  systemctl start  etcd
  
  #开机自启
  systemctl enable etcd
  
  ```
 ## 健康检查
 
 ```
 #不携带证书访问不了
 root@ubuntu-Vostro-3668:/opt/etcd/bin# ./etcdctl   member list
{"level":"warn","ts":"2020-04-09T12:57:15.536+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"endpoint://client-4d2e6b5c-0b51-4568-9e06-ce9a5a88b82b/127.0.0.1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest connection error: connection closed"}
Error: context deadline exceeded
 #携带证书可以正常访问
root@ubuntu-Vostro-3668:/opt/etcd/bin# ./etcdctl --cacert=../ssl/ca.pem  --cert=../ssl/etcd.pem  --key=../ssl/etcd-key.pem  member list
326a8acae263a159, started, etcd2, https://192.168.0.121:2380, https://127.0.0.1:2379,https://192.168.0.121:2379, false
91d478e5fe6d6d92, started, etcd1, https://192.168.0.112:2380, https://127.0.0.1:2379,https://192.168.0.112:2379, false
d8ffe8217780058e, started, etcd3, https://192.168.0.149:2380, https://127.0.0.1:2379,https://192.168.0.149:2379, false

 
 ```
 
## 参考资料

**[etcd配置项说明](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/configuration.md)**


**[etcd集群搭建]（https://blog.csdn.net/deep_kang/article/details/90055395）**
