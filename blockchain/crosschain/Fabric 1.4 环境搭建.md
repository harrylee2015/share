# Frabic1.4开发环境部署
这里以版本1.4.3为例

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
git checkout release-1.3
# 如果go版本高于1.13的话，可以关掉mod功能，采用vendor管理
export GO15VENDOREXPERIMENT=1  #开启vendor版本管理
export GO111MODULE="off"       #关闭新版本的go mod管理
make release
```

## 将编译好的bin文件夹整个copy到fabric-samples项目下

```
 cd release/linux-amd64/
 
 mv bin   gopath/src/github.com/hyperledger/fabric-samples/
```

## 修改生成证书配置

crypto-config.yaml
```


# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

# ---------------------------------------------------------------------------
# "OrdererOrgs" - Definition of organizations managing orderer nodes
# ---------------------------------------------------------------------------
OrdererOrgs:
  # ---------------------------------------------------------------------------
  # Orderer
  # ---------------------------------------------------------------------------
  - Name: Orderer
    Domain: example.com
    EnableNodeOUs: true
    # ---------------------------------------------------------------------------
    # "Specs" - See PeerOrgs below for complete description
    # ---------------------------------------------------------------------------
    Specs:
      - Hostname: orderer
        SANS:
         - "localhost"
         - "127.0.0.1"

# ---------------------------------------------------------------------------
# "PeerOrgs" - Definition of organizations managing peer nodes
# ---------------------------------------------------------------------------
PeerOrgs:
  # ---------------------------------------------------------------------------
  # Org1
  # ---------------------------------------------------------------------------
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true
    # ---------------------------------------------------------------------------
    # "Specs"
    # ---------------------------------------------------------------------------
    # Uncomment this section to enable the explicit definition of hosts in your
    # configuration.  Most users will want to use Template, below
    #
    # Specs is an array of Spec entries.  Each Spec entry consists of two fields:
    #   - Hostname:   (Required) The desired hostname, sans the domain.
    #   - CommonName: (Optional) Specifies the template or explicit override for
    #                 the CN.  By default, this is the template:
    #
    #                              "{{.Hostname}}.{{.Domain}}"
    #
    #                 which obtains its values from the Spec.Hostname and
    #                 Org.Domain, respectively.
    # ---------------------------------------------------------------------------
    # Specs:
    #   - Hostname: foo # implicitly "foo.org1.example.com"
    #     CommonName: foo27.org5.example.com # overrides Hostname-based FQDN set above
    #   - Hostname: bar
    #   - Hostname: baz
    # ---------------------------------------------------------------------------
    # "Template"
    # ---------------------------------------------------------------------------
    # Allows for the definition of 1 or more hosts that are created sequentially
    # from a template. By default, this looks like "peer%d" from 0 to Count-1.
    # You may override the number of nodes (Count), the starting index (Start)
    # or the template used to construct the name (Hostname).
    #
    # Note: Template and Specs are not mutually exclusive.  You may define both
    # sections and the aggregate nodes will be created for you.  Take care with
    # name collisions
    # ---------------------------------------------------------------------------
    Template:
      Count: 2
      SANS:
         - "localhost"
         - "127.0.0.1"   
      # Start: 5
      # Hostname: {{.Prefix}}{{.Index}} # default
    # ---------------------------------------------------------------------------
    # "Users"
    # ---------------------------------------------------------------------------
    # Count: The number of user accounts _in addition_ to Admin
    # ---------------------------------------------------------------------------
    Users:
      Count: 1
  # ---------------------------------------------------------------------------
  # Org2: See "Org1" for full specification
  # ---------------------------------------------------------------------------
  - Name: Org2
    Domain: org2.example.com
    EnableNodeOUs: true
    Template:
      Count: 2
      SANS:
         - "localhost"
         - "127.0.0.1"
    Users:
      Count: 1

```

## 修改docker-compose-base.yaml

```
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

services:

  orderer.example.com:
    container_name: orderer.example.com
    extends:
      file: peer-base.yaml
      service: orderer-base
    volumes:
        - ../channel-artifacts/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
        - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
        - ../crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls
        - orderer.example.com:/var/hyperledger/production/orderer
    ports:
      - 7050:7050

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org1.example.com:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org1.example.com:8051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
        - /var/run/docker.sock:/host/var/run/docker.sock
        - ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls:/etc/hyperledger/fabric/tls
        - peer0.org1.example.com:/var/hyperledger/production
    ports:
      - 7051:7051
      - 7052:7052
      - 7053:7053

  peer1.org1.example.com:
    container_name: peer1.org1.example.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer1.org1.example.com
      - CORE_PEER_ADDRESS=peer1.org1.example.com:8051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:8051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org1.example.com:8052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:8052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org1.example.com:8051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
    volumes:
        - /var/run/docker.sock:/host/var/run/docker.sock
        - ../crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/org1.example.com/peers/peer1.org1.example.com/tls:/etc/hyperledger/fabric/tls
        - peer1.org1.example.com:/var/hyperledger/production

    ports:
      - 8051:8051
      - 8052:8052
      - 8053:8053

  peer0.org2.example.com:
    container_name: peer0.org2.example.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer0.org2.example.com
      - CORE_PEER_ADDRESS=peer0.org2.example.com:9051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:9051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org2.example.com:9052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:9052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org2.example.com:9051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer1.org2.example.com:10051
      - CORE_PEER_LOCALMSPID=Org2MSP
    volumes:
        - /var/run/docker.sock:/host/var/run/docker.sock
        - ../crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls:/etc/hyperledger/fabric/tls
        - peer0.org2.example.com:/var/hyperledger/production
    ports:
      - 9051:9051
      - 9052:9052
      - 9053:9053

  peer1.org2.example.com:
    container_name: peer1.org2.example.com
    extends:
      file: peer-base.yaml
      service: peer-base
    environment:
      - CORE_PEER_ID=peer1.org2.example.com
      - CORE_PEER_ADDRESS=peer1.org2.example.com:10051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:10051
      - CORE_PEER_CHAINCODEADDRESS=peer1.org2.example.com:10052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:10052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer1.org2.example.com:10051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org2.example.com:9051
      - CORE_PEER_LOCALMSPID=Org2MSP
    volumes:
        - /var/run/docker.sock:/host/var/run/docker.sock
        - ../crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/msp:/etc/hyperledger/fabric/msp
        - ../crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls:/etc/hyperledger/fabric/tls
        - peer1.org2.example.com:/var/hyperledger/production
    ports:
      - 10051:10051
      - 10052:10052
      - 10053:10053

```

## 初始化相关配置

```
cd gopath/src/github.com/hyperledger/fabric-samples/first-network/

./byfn.sh generate 

```

## 启动服务

```
./byfn.sh up  -i 1.4.3   #用1.4.3镜像，默认采用solo共识

```

## 进入cli容器，进行命令行交互式操作

```
docker exec -it cli bash

#需要注意的是当需要部署合约时，需要将合约下载到cli容器中，默认已经安装了相关的go环境，版本时1.12 ，
#需要其他依赖包时，需要开启mod功能

export GO111MODULE=on

export GOPROXY=https://goproxy.cn,direct

#部署broker合约

peer chaincode install -n broker  -v 1.0 -l golang -p  gitlab.33.cn/link33/broker/

#部署data_swapper合约
peer chaincode install -n data_swapper  -v 1.0 -l golang -p  gitlab.33.cn/link33/data_swapper

```
## 在宿主机外与docker-compose启动的fabric进行交互

导入环境变量

```
export CORE_PEER_TLS_KEY_FILE=gopath/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.key
export CORE_PEER_LOCALMSPID=Org1MSP
export FABRIC_CFG_PATH=gopath/src/github.com/hyperledger/fabric-samples/config/fabric
export CORE_PEER_TLS_ENABLED=true
export ORDERER_CA=gopath/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export CORE_PEER_TLS_CERT_FILE=gopath/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt
export CORE_PEER_ID=cli
export CORE_PEER_MSPCONFIGPATH=gopath/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp

export CORE_PEER_TLS_ROOTCERT_FILE=gopath/src/github.com/hyperledger/fabric-samples/first-network/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=localhost:7051


```
 
## 安装链码

```

harry@harry-VirtualBox:~/gopath/src/gitlab.33.cn/link33$  /home/harry/fabric/fabric-samples/bin/peer chaincode install -n broker  -v 1.0 -l golang -p gitlab.33.cn/link33/broker
2021-11-29 14:50:54.763 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2021-11-29 14:50:54.763 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2021-11-29 14:50:55.207 CST [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 
harry@harry-VirtualBox:~/gopath/src/gitlab.33.cn/link33$  /home/harry/fabric/fabric-samples/bin/peer chaincode install -n data_swapper  -v 1.0 -l golang -p gitlab.33.cn/link33/data_swapper
2021-11-29 15:08:07.678 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2021-11-29 15:08:07.678 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2021-11-29 15:08:07.914 CST [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 


```


## 链码初始化及调用示例

```
# 初始化合约

/home/harry/fabric/fabric-samples/bin/peer chaincode instantiate -o localhost:7050 --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n data_swapper  -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"

# 设置值
harry@harry-VirtualBox:~/gopath/src/gitlab.33.cn/link33$  /home/harry/fabric/fabric-samples/bin/peer chaincode invoke   -o localhost:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n data_swapper  -c '{"Args":["set","111111","11111"]}'
2021-11-29 15:16:52.699 CST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200 


# 查询值

harry@harry-VirtualBox:~/gopath/src/gitlab.33.cn/link33$ /home/harry/fabric/fabric-samples/bin/peer chaincode query   -o localhost:7050  --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n data_swapper  -c '{"Args":["get","111111"]}'
11111

```

## 宿主机外客户端 sdk与docker环境交互，只需配置一下宿主机的hosts

```
比如：各个容器对应的ip可以通过cli容器安装ping工具包，进行探测

docker exec -it cli bash

apt-get install inetutils-ping

....

172.22.0.4 peer1.org2.example.com
172.22.0.6 orderer.example.com
172.22.0.2 peer0.org2.example.com
172.22.0.3 peer0.org1.example.com
172.22.0.5 peer1.org1.example.com


```
