# fabric 1.4跨链合约的部署及使用

首先我们需完成[fabric1.4的环境搭建工作](Fabric 1.4环境搭建)

## 合约部署（链码安装)

* 节点peer0部署合约

导入环境变量

```
export CORE_PEER_TLS_KEY_FILE=/home/harry/fabric/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.key
export CORE_PEER_LOCALMSPID=Org2MSP
export FABRIC_CFG_PATH=/home/harry/fabric/fabric-samples/config/fabric
export CORE_PEER_TLS_ENABLED=true
export ORDERER_CA=/home/harry/fabric/fabric-samples/first-network/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
export CORE_PEER_TLS_CERT_FILE=/home/harry/fabric/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/server.crt
export CORE_PEER_ID=cli
export CORE_PEER_MSPCONFIGPATH=/home/harry/fabric/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp

export CORE_PEER_TLS_ROOTCERT_FILE=/home/harry/fabric/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_ADDRESS=peer0.org2.example.com:9051

```
* 部署broker合约

```
git clone  git@gitlab.33.cn:link33/broker.git

cd  $GOPATH/src/gitlab.33.cn:link33/broker/

/home/harry/fabric/fabric-samples/bin/peer  chaincode install -n broker  -v 1.0  -l  golang  -p  gitlab.33.cn/link33/broker

```
正确部署会有如下日志

```
2021-12-17 15:04:21.974 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2021-12-17 15:04:21.975 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2021-12-17 15:04:29.116 CST [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 
```

* 部署data_swapper合约

```
git clone  git@gitlab.33.cn:link33/data_swapper.git

cd  $GOPATH/src/gitlab.33.cn:link33/data_swapper/

/home/harry/fabric/fabric-samples/bin/peer chaincode install -n data_swapper  -v 1.0 -l golang -p gitlab.33.cn/link33/data_swapper

```
正确部署会有如下日志

```
2021-12-17 15:21:01.826 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2021-12-17 15:21:01.827 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
2021-12-17 15:21:02.726 CST [chaincodeCmd] install -> INFO 003 Installed remotely response:<status:200 payload:"OK" > 
```
* peer1部署合约

切换环境变量

```
export CORE_PEER_ADDRESS=peer1.org2.example.com:10051
export CORE_PEER_TLS_ROOTCERT_FILE=/home/harry/fabric/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/ca.crt
export CORE_PEER_TLS_CERT_FILE=/home/harry/fabric/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.crt
export CORE_PEER_TLS_KEY_FILE=/home/harry/fabric/fabric-samples/first-network/crypto-config/peerOrganizations/org2.example.com/peers/peer1.org2.example.com/tls/server.key

```
然后重复上面，broker和data_swapper合约的部署过程，每个节点都需要单独安装链码

* 初始化broker合约

```
/home/harry/fabric/fabric-samples/bin/peer chaincode instantiate -o orderer.example.com:7050   --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n broker -v 1.0 -c '{"Args":["initialize","link33","fabric"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"

```
成功初始化会有如下日志

```
2021-12-17 15:28:13.875 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 001 Using default escc
2021-12-17 15:28:13.875 CST [chaincodeCmd] checkChaincodeCmdParams -> INFO 002 Using default vscc
```

* 初始化data_swapper合约

```
/home/harry/fabric/fabric-samples/bin/peer chaincode instantiate -o orderer.example.com:7050   --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n data_swapper  -v 1.0 -c '{"Args":["init"]}' -P "OR ('Org1MSP.peer','Org2MSP.peer')"
```
有可能在部署的合约的过程中会报错啥的，这时不要慌，看看错误日志一般就知道怎么解决了




## 合约的调用

* 注册data_swapper合约

```
/home/harry/fabric/fabric-samples/bin/peer chaincode invoke -o orderer.example.com:7050   --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n data_swapper  -c '{"Args":["register"]}'
```
正确注册会报如下日志

```
2021-12-20 10:46:36.610 CST [chaincodeCmd] chaincodeInvokeOrQuery -> INFO 001 Chaincode invoke successful. result: status:200 payload:"mychannel&data_swapper"
```

* 审批data_swapper合约

```
/home/harry/fabric/fabric-samples/bin/peer chaincode invoke -o orderer.example.com:7050   --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n broker -c '{"Args":["audit","mychannel","data_swapper","1"]}'

```

## 测试

* 设置K,V

```
/home/harry/fabric/fabric-samples/bin/peer chaincode invoke -o orderer.example.com:7050   --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n data_swapper  -c '{"Args":["set","v2","version2"]}'

```

* 查询

```
harry@harry-VirtualBox:~/gopath/src/gitlab.33.cn/link33/data_swapper$ /home/harry/fabric/fabric-samples/bin/peer chaincode query    -C mychannel -n data_swapper -c '{"Args":["get","v2"]}'
version2

```

以上合约部署没有问题


## 跨链请求操作示例

此步操作前提是完成[基础环境的搭建](Sidecar使用文档)，chain33,fabric,sidecar环境都已经构建好，正常运行

fabric上面的跨链请求本质上面是从其他应用链获取数据，写入到fabric上面的过程

假设，我们chain33应用链上面有key=v1的值，value=version1

我们可以在fabric上面发送一个跨链请求

```
/home/harry/fabric/fabric-samples/bin/peer chaincode invoke -o orderer.example.com:7050   --tls $CORE_PEER_TLS_ENABLED --cafile $ORDERER_CA  -C mychannel -n data_swapper  -c '{"Args":["get","link33:chain33:local&broker","v1"]}'

```

正确跨链，跨链插件会有如下日志

```
INFO[2021-12-20T15:08:19.302] Monitor started                               module=app
INFO[2021-12-20T15:08:19.302] start p2p success                             module=lightp2p
2021-12-20T15:08:19.307+0800 [DEBUG] plugin.sh: 2021-12-20T15:08:19.306+0800 [INFO ] fabric-client: #########################GetIBTP
INFO[2021-12-20T15:08:19.313] Connect successfully                          address:=0x72e70ccF0b4c93ce2354F107425cF229590827F3 module=peer_mgr node=QmanEFULPJ3oiLQerCw9WVNxVasaLRXRwUeb4nQ7KEskmH
2021-12-20T15:09:03.326+0800 [DEBUG] plugin.sh: 2021-12-20T15:09:03.326+0800 [INFO ] fabric-client: #######################get ibtp,From:: link33:fabric:mychannel&data_swapper=link33:chain33:local&broker
INFO[2021-12-20T15:09:04.006] Apply tx                                      from="link33:fabric:mychannel&data_swapper" id="link33:fabric:mychannel&data_swapper-link33:chain33:local&broker-1" index=1 module=app type=RECEIPT_SUCCESS
2021-12-20T15:09:05.404+0800 [DEBUG] plugin.sh: 2021-12-20T15:09:05.403+0800 [INFO ] fabric-client: response: cc status=200 payload={"ok":true,"message":"","data":null}
INFO[2021-12-20T15:09:05.404] Execute callback                              fields.msg= index=1 module=app status=true type=RECEIPT_SUCCESS

```

* 查询以下结果，看是否跨链成功

```
harry@harry-VirtualBox:~/gopath/src/gitlab.33.cn/link33/data_swapper$ /home/harry/fabric/fabric-samples/bin/peer chaincode query    -C mychannel -n data_swapper -c '{"Args":["get","v1"]}'

version1

```
可以看到数据已经写入成功
