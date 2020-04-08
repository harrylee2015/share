# 部署带本地自签CA认证的etcd集群

## 前言

  etcd在生产环境中部署时往往都是采用https方式进行访问，这往往需要我们本地搞个自签证书，下面我们一起来学习一下如何去生成和部署


## 下载所需的包(cfssl,生成证书工具)

```
mkdir /usr/local/src/etcd/ 
cd /usr/local/src/etcd/
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
mkdir /usr/local/src/etcd/ssl
cd /usr/local/src/etcd/ssl

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
      "O": "etcd",
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
      "O": "etcd",
      "OU": "System"
    }
  ]
}

如果 hosts 字段不为空则需要指定授权使用该证书的 IP 或域名列表，由于该证书后续被 etcd 集群使用，所以填写IP即可。
因为本次部署etcd是单台，那么则需要填写单台的IP地址即可。
       
#生成 etcd证书和私钥 
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=etcd etcd-csr.json | cfssljson -bare etcd

#将TLS 认证文件拷贝至证书目录下

mkdir -p /etc/etcd/etcdSSL

cp * /etc/etcd/etcdSSL

```


**[etcd配置项说明](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/configuration.md)**
