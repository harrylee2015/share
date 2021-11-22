# frabic 开发环境部署
这里以版本1.4.4为例

## 环境准备
 * [准备工作](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html)
  
 * ubuntu 16.04 操作系统
 
 * go 1.11.x 版本以上
 
 * git 
 
 * [docker && docker-compose](https://hyperledger-fabric.readthedocs.io/en/latest/prereqs.html#docker-and-docker-compose)


## 下载fabric-sample项目，并切换到 1.4 版本

```
git clone https://github.com/hyperledger/fabric-samples.git

git checkout -b release-1.4  origin/release-1.4
```

## 下载或者编译fabric1.4 

```
git clone "https://github.com/hyperledger/fabric.git"
cd fabric/
#本文使用的是1.4版本的Fabric，需要以下命令检出fabric版本为1.4的分支
git checkout release-1.4
# 如果go版本高于1.13的话，可以关掉mod功能，采用vendor管理
#export GO15VENDOREXPERIMENT=1  #开启vendor版本管理
#export GO111MODULE="off"       #关闭新版本的go mod管理
make release
```

## 将编译好的bin文件夹整个copy到fabric-samples项目下

```
 cd release/linux-amd64/
 
 mv bin   gopath/src/github.com/hyperledger/fabric-samples/
```

## 初始化相关配置

```
cd gopath/src/github.com/hyperledger/fabric-samples/first-network/

./byfn.sh generate 

```

## 启动服务

```
./byfn.sh up  -i 1.4.4   #用1.4.4镜像，默认采用etcdraft共识

```

## 进入cli容器，进行命令行交互式操作

```
docker exec -it cli bash

#需要注意的是当需要部署合约时，需要将合约下载到cli容器中，默认已经安装了相关的go环境，版本时1.12 ，
#需要其他依赖包时，需要开启mod功能

export GO111MODULE=on

export GOPROXY=https://goproxy.cn,direct

#部署broker合约

peer chaincode install -n broker  ^C 1.0 -l golang -p  gitlab.33.cn/link33/sidecar-client-fabric/example/contracts/src/broker/

#部署data_swapper合约
peer chaincode install -n data_swapper  -v 1.0 -l golang -p  gitlab.33.cn/link33/sidecar-client-fabric/example/contracts/src/data_swapper

```

 
