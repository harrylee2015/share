# sidecar使用文档

## 准备工作

如果你想在本地或者服务器上运行跨链网关，你需要在机器上安装有以下的依赖：
[golang1.13+](https://go.dev/doc/install)、[packr](https://github.com/gobuffalo/packr)


## 安装和配置

执行下面的命令进行编译sidecar

```
export CGO_ENABLED=1  //开启CGO编译功能，sidecar中依赖库会用到c语言库

git clone https://gitlab.33.cn/link33/sidecar

cd sidecar

make 

```

## 创建工作文件sidecar

```
mkdir  -p $HOME/sidecar/plugins

// 将上面编译好的可执行文件放入创建的文件夹下
mv  bin/sidecar   $HOME/sidecar/
```

## fabric插件环境配置

* [Fabric1.4环境搭建](Fabric1.4环境搭建)

* [Fabric1.4跨链合约的部署及使用](Fabric1.4跨链合约的部署及使用)

* [Fabric1.4插件配置](Fabric1.4插件配置)


## chain33插件环境配置

* [Chain33开发环境搭建](Chain33开发环境搭建)

* [Chain33跨链合约初始化及使用](Chain33中broker合约的初始化及cli命令操作)

* [Chain33插件配置](Chain33插件配置)


## 跨链网关配置

sidecar想要完成运行起来需要配置中继链和应用链的相关信息，可以指定配置目录，默认路径为 $HOME/sidecar
也可以导入环境变量:
```
export SIDECAR_PATH=$HOME/sidecar  # 默认路径
 
or

export  SIDECAR_PATH=/home/harry/distribute_sidecar/sidecar01/  #指定的配置目录
```

## 初始化网关节点

```
./sidecar init repo=$SIDECAR_PATH

```
成功初始化后，系统会自动生成如下目录文件:

```
harry@harry-VirtualBox:~/sidecar$ ls -l
总用量 37664
-rw-r--r-- 1 harry harry       25 12月 15 10:39 api
-rw-r--r-- 1 harry harry      250 12月 15 10:39 authz_model.conf
-rw-r--r-- 1 harry harry      220 12月 15 10:39 authz_policy.csv
drwxr-xr-x 2 harry harry     4096 12月 15 10:39 certs
-rw-rw-r-- 1 harry harry      195 12月 15 10:39 key.json
-rw-rw-r-- 1 harry harry      227 12月 15 10:39 node.priv
-rwxrwxr-x 1 harry harry 38531376 12月 15 10:37 sidecar
-rw-r--r-- 1 harry harry      970 12月 15 10:39 sidecar.toml
```

## 获取网关节点ID

```
./sidecar p2p id
```
正确会显示如下:
```
harry@harry-VirtualBox:~/sidecar$ ./sidecar  p2p id
QmXtgbubrdPQHoiXSTibb4xvTL4QVPykuKW73HZcdH9f7k

```
## 修改sidecar.toml文件配置

单节点网关下同时运行fabric和chain33插件配置示例:

```

RepoRoot = "/home/harry/distribute_sidecar/sidecar01/"
title = "Sidecar"

[port]
http = 44744
pprof = 44755

[log]
dir = "logs"
filename = "sidecar.log"
ReportCaller = false
level = "info"
[log.module]
api_server = "info"
appchain_mgr = "info"
App = "info"
exchanger = "info"
executor = "info"
monitor = "info"
peer_mgr = "info"
router = "info"
swarm = "info"
syncer = "info"
manager = "info"
rule = "info"

[[appchains]]
enable = true
type = "appchain"
did = "link33:chain33:local&broker"
config = "chain33"
plugin = "chain33-client"


[[appchains]]
enable = true
type = "appchain"
did = "link33:fabric:mychannel&data_swapper"
plugin = "fabric-client-1.4"
config = "fabric"

[security]
EnableTLS = false
tlsca = ""
CommonName = ""

[peer]
# peer地址使用上面获取的p2p节点地址
peers = ["/ip4/127.0.0.1/tcp/4000/p2p/QmXtgbubrdPQHoiXSTibb4xvTL4QVPykuKW73HZcdH9f7k"]
connectors = ["localhost:60011"]
providers = 1

```

## 启动跨链网关sidecar

```
./sidecar start repo=$SIDECAR_PATH

```
