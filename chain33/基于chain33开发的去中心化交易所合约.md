

# 基于chain33开发的去中心化交易所合约

## 背景

  目前chain33官方plugin下面已经有类似的去中心化交易合约trade,但是在实际使用的过程中存在诸多问题。
  
  序号|问题
  ---|---
  1|根据挂单号买卖，不能自动撮合交易
  2| rpc查询接口,不全，不能仅根据交易对信息，获取挂单列表，需要额外的数据库去支持，繁琐
  3|构建的交易参数众多，不太好用
  
  
## 需求分析
  
  需要新构建一个新的合约，解决如上问题，实现类似如传统中心化交易所那样的便捷性
  
  序号|难点
  ---|---
  1|交易数据模型制定
  2|账户状态切换
  3|多方位的查询接口，交易对，市场深度，单号查询等
  4|利用levelDB中list功能，设计先进先出的队列

查询接口需求

 1. 提供指定交易对的市场深度查询
 
 2. 提供指定交易对挂单查询(系统内部使用）
 
 3. 提供根据挂单状态查询用户自己最新的挂单信息

 4. 查询最新已经成交的挂单信息
 

## 存储模型设计

 ```
message ExchangeAction {
    oneof value {
        LimitOrder limitOrder = 1;
        MarketOrder marketOrder = 2;
        RevokeOrder revokeOrder = 3;
    }
    int32 ty = 6;
}
//限价订单
message LimitOrder {
    //交易对
    asset leftAsset  = 1;
    //交易对
    asset rightAsset  = 2;
    //价格
    double price  = 3;
    //总量
    int64 amount = 4;
    //操作， 1为买，2为卖
    int32 op = 5;
}

 //市价委托
message MarketOrder {
    //资产1
    asset leftAsset  = 1;
    //资产2
    asset rightAsset = 2;
    //总量
    int64 amount = 3;
    //操作， 1为买，2为卖
    int32 op = 4;
}

 //撤回订单
message RevokeOrder {
    //订单号
    int64 orderID   = 1;
}
//资产类型
message asset {
    string execer = 1;
    string symbol = 2;
}

 //订单信息
message Order {
    int64 orderID   = 1;
    oneof value {
        LimitOrder limitOrder = 2;
        MarketOrder marketOrder = 3;
    }
    //挂单类型
    int32 ty = 4;
    //已经成交的数量
    int64 executed = 5;
    //余额
    int64  balance = 6;
    //状态,0 挂单中ordered， 1 完成completed， 2撤回 revoked
    int32 status = 7;
    //用户地址
    string addr = 8;
    //更新时间
    int64 updateTime = 9;
    //索引
    int64 index = 10;
}

 //查询接口
message QueryMarketDepth {
    //资产1
    asset leftAsset  = 1;
    //资产2
    asset rightAsset = 2;
    //操作， 1为买，2为卖
    int32 op = 3;
    // 这里用价格作为索引值
    string primaryKey  = 4;
    //单页返回多少条记录，默认返回10条,为了系统安全最多单次只能返回20条
    int32 count = 5;
}
//市场深度
message MarketDepth {
    //资产1
    asset leftAsset  = 1;
    //资产2
    asset rightAsset = 2;
    //价格
    double price  = 3;
    //总量
    int64 amount = 4;
    //操作， 1为买，2为卖
    int32 op = 5;
}
//查询接口返回的市场深度列表
message MarketDepthList {
    repeated MarketDepth list = 1;
    string primaryKey = 2;
}

 //查询最新得成交信息,外部接口
message QueryCompletedOrderList {
    //资产1
    asset leftAsset  = 1;
    //资产2
    asset rightAsset = 2;
    // 索引值
    string primaryKey = 3;
    //单页返回多少条记录，默认返回10条,为了系统安全最多单次只能返回20条
    int32 count = 4;
    // 0降序，1升序，默认降序
    int32 direction = 5;
}

 //根据orderID去查询订单信息
message QueryOrder {
    int64 orderID   = 1;
}
//根据地址，状态查询用户自己的挂单信息
message QueryOrderList {
    //挂单状态必填(默认是0,只查询ordered挂单中的)
    int32 status = 1;
    //用户地址信息，必填
    string address = 2;
    // 主键索引
    string primaryKey = 3;
    //单页返回多少条记录，默认返回10条,为了系统安全最多单次只能返回20条
    int32 count = 4;
    // 0降序，1升序，默认降序
    int32 direction = 5;
}
//订单列表
message OrderList {
    repeated Order list = 1;
    string primaryKey = 2;
}


 //exchange执行票据日志
message ReceiptExchange {
    Order order = 1;
    repeated Order matchOrders = 2;
    int64  index      = 3;
}
 ```
 [具体proto文件](https://github.com/harrylee2015/plugin/blob/exchange/plugin/dapp/exchange/proto/exchange.proto)

## 索引设计

 |键|值|用途|说明|
|-|-|-|-|
|~~LODB-exchange-depth-{leftAsset}-{rightAsset}-{op}:{price}~~|资产交易对买/卖的深度|给用户提供查询，用于实时查询市场深度|变参为leftAsset，rightAsset，op (1为买，2为卖),价格(%0016 占16位,根据价格进行计算得来)|
|~~LODB-exchange-order-{leftAsset}-{rightAsset}-{op}:{price}:{index}~~|挂单ID和index|记录具体的挂单orderID和index|变参为leftAsset，rightAsset，op (1为买，2为卖),价格,和index(%022d 比系统原有的多占了四位用于撮合时,根据顺序来区分和设置唯一的key)|
|~~LODB-exchange-completed-{leftAsset}-{rightAsset}-{index}~~|挂单ID和index|记录已经成交挂单的orderID和最新index|变参为leftAsset，rightAsset，op index(%022d 多了四位用于撮合时,根据顺序来区分和设置唯一的key)|
|~~LODB-exchange-addr:{addr}:{status}:{index}~~|挂单ID和index|根据地址和状态记录挂单的orderID和index，用于用户根据挂单状态自己地址下面的挂单详情|变参为addr，status(0 ordered,1 completd,2 revoked)，index (%022d 比系统原有的多占了四位用于撮合时,根据顺序来区分和设置唯一的key)|
|mavl-exchange-orderID:{orderID}|挂单信息|存在状态数据库中，任何人都可以根据具体orderID来查询具体的挂单信息|变参为orderID(~~也就是交易的txhash~~)变为int64，由系统更具区块高度和index自动生成|
全改为table方式实现

[合约代码中的具体实现](https://github.com/harrylee2015/plugin/blob/exchange/plugin/dapp/exchange/executor/tables.go)




## 查询接口设计


查询方法名称|功能
-----|------
QueryMarketDepth|获取指定交易资产的市场深度
QueryCompletedOrderList|实时获取指定交易对最新的成交信息
QueryOrder|根据orderID订单号查询具体的订单信息
QueryOrderList|根据用户地址和订单状态（ordered,completed,revoked)，实时地获取相应相应的订单详情

[代码中的实现](https://github.com/harrylee2015/plugin/blob/exchange/plugin/dapp/exchange/executor/query.go)

 


## exchange合约执行流程

![exchange合约执行流程图](./resource/exchange.png)


## 注意事项
 
合约撮合规则如下：

 序号|规则
---|----
1|买家获利得原则
2|买单高于市场价，按价格由低往高撮合
3|卖单低于市场价，按价格由高往低进行撮合
4|价格相同按先进先出的原则进行撮合
5|出于系统安全考虑，最大撮合深度为100单，单笔挂单最小为1e8,就是一个bty



## 遗留问题

 ~~1. 高于市场挂单,应该由按价格由低往高进行撮合~~
  
  ~~2. 低于市场挂单,应该按价格由高往低进行撮合~~
