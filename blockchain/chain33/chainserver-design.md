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

**主链配置信息表chainserver_mainchain**

字段|类型|说明
---|---|--
id |interger(11)|主键ID
uid|varchar(32)|用户UID
order_id|varchar(32)|订单编号
consensus_type|int(1)|0:联盟链 1:私有链 2:平行链
product_type|int(1)|产品类型 0 区块链  1区块链+云服务器
deploy_type|int(1)|0:单机部署 1:集群部署
title|varchar(32)|链名称
node_num|int(11)|节点数量
init_inner_ip|varchar(512)|初始化集群时,节点内部ip,用“，”号分割
init_outer_ip|varchar(512)|初始化集群时,节点外部IP,用“，”号分割
history_inner_ip|varchar(1024)|历史节点记录,只增不删
current_inner_ip|varchar(1024)|当前的节点内网ip,增删都执行
history_outer_ip|varchar(1024)|历史节点记录,只增不删
current_outer_ip|varchar(1024)|当前的节点ip,增删都执行
super_manager_addr|varchar(128)|管理员地址
block_size|int(10)|区块大小
block_time_out|int(10)|出块间隔，超时时间
jrpc_bind_port|int|jrpc端口，默认8801
grpc_bind_port|int|grpc端口，默认8802
params|varchar(1024)|预留配置参数
chain_version|varchar(32)|链的二进制版本包
chain_url|varchar(512)|模版本下载地址
download_url|varchar(1500)|配置文件包生成地址
detail|varchar(10240)|操作详细信息当部署失败时，存储具体的失败原因
create_time|bigint(20)|创建时间
update_time|bigint(20)|更新时间



**平行链配置信息表chainserver_parachain**

字段|类型|说明
---|---|--
id |interger(11)|主键ID
uid|varchar(32)|用户UID
mainchain_id|interger(11)|主链ID
order_id|varchar(32)|订单编号
consensus_type|int(1)|0:联盟链 1:私有链 2:平行链
product_type|int(1)|产品类型 0 区块链  1区块链+云服务器
deploy_type|int(1)|0:单机部署 1:集群部署
title|varchar(32)|链名称
node_num|int(11)|节点数量
init_inner_ip|varchar(512)|初始化集群时,节点内部ip,用“，”号分割
init_outer_ip|varchar(512)|初始化集群时,节点外部IP,用“，”号分割
history_inner_ip|varchar(1024)|历史节点记录,只增不删
current_inner_ip|varchar(1024)|当前的节点内网ip,增删都执行
history_outer_ip|varchar(1024)|历史节点记录,只增不删
current_outer_ip|varchar(1024)|当前的节点ip,增删都执行
title|varchar(128)|平行链名称
super_manager_addr|varchar(128)|管理员地址
jrpc_bind_port|int|jrpc端口，默认8901
grpc_bind_port|int|grpc端口，默认8902
params|varchar(1024)|预留配置参数
chain_version|varchar(32)|链的二进制版本包
chain_url|varchar(512)|模版本下载地址
download_url|varchar(1500)|配置文件包生成地址
detail|varchar(10240)|操作详细信息当部署失败时，存储具体的失败原因
create_time|bigint(20)|创建时间
update_time|bigint(20)|更新时间

**节点信息表**

**平行链配置信息表chainserver_parachain**

字段|类型|说明
---|---|--
id |interger(11)|主键ID
uid|varchar(32)|用户UID
mainchain_id|interger(11)|主链ID
order_id|varchar(32)|订单编号
consensus_type|int(1)|0:联盟链 1:私有链 2:平行链
product_type|int(1)|产品类型 0 区块链  1区块链+云服务器
deploy_type|int(1)|0:单机部署 1:集群部署
title|varchar(32)|链名称
node_num|int(11)|节点数量
init_inner_ip|varchar(512)|初始化集群时,节点内部ip,用“，”号分割
init_outer_ip|varchar(512)|初始化集群时,节点外部IP,用“，”号分割
history_inner_ip|varchar(1024)|历史节点记录,只增不删
current_inner_ip|varchar(1024)|当前的节点内网ip,增删都执行
history_outer_ip|varchar(1024)|历史节点记录,只增不删
current_outer_ip|varchar(1024)|当前的节点ip,增删都执行
title|varchar(128)|平行链名称
super_manager_addr|varchar(128)|管理员地址
jrpc_bind_port|int|jrpc端口，默认8901
grpc_bind_port|int|grpc端口，默认8902
params|varchar(1024)|预留配置参数
chain_version|varchar(32)|链的二进制版本包
chain_url|varchar(512)|模版本下载地址
download_url|varchar(1500)|配置文件包生成地址
detail|varchar(10240)|操作详细信息当部署失败时，存储具体的失败原因
create_time|bigint(20)|创建时间
update_time|bigint(20)|更新时间