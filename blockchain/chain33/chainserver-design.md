# chainserver-design

## URI接口设计

 功能|URI
 ----|-----
 创建管理员|chainserver/manager/create
 初始化tendermint区块链|chainserver/tendermint/init
 初始化raft     |chainserver/raft/init
 初始化para     |chainserver/para/init
 添加tendermint节点|chainserver/tendermint/add
 添加raft节点|chainserver/raft/add
 添加para节点|chainserver/para/add
 删除tendermint节点|chainserver/tendermint/delete
 删除raft节点|chainserver/raft/delete
 删除para节点|chainserver/para/delete
 
 ## 表设计
 
**管理员信息表chainserver_manager**

 字段|类型|说明
 ---|---|---
 id |interger(11)|主键ID
 uid|varchar(32)|用户UID
 mid|varchar(32)|管理员ID,必须唯一
 name|varchar(64)|管理员名称
 addr|varchar(128)|管理员地址
 pubkey|varchar(256)|管理员公钥
 privkey|varchar(256)|管理员私钥
 status|tinyint(4)|管理员状态，0正常，1删除
 create_time|bigint(20)|新建时间
 update_time|bigint(20)|更新时间
