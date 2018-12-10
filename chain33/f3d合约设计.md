## 1.1. 存储设计

### 1.1.1. 业务模型

**游戏轮次信息：**
```protobuf
message RoundInfo {
    // 游戏轮次
    int64 round = 1;
    // 本轮游戏开始事件
    int64 beginTime  = 2;
    // 本轮游戏结束时间
    int64 endTime    = 3;
    // 本轮游戏目前为止最后一把钥匙持有人（游戏开奖时，这个就是中奖人）
    string lastOwner   = 4;
    // 最后一把钥匙的购买时间
    int64 lastKeyTime = 5;
    // 最后一把钥匙的价格
    int64 lastKeyPrice = 6;
    // 本轮游戏奖金池总额
    int64 bonusPool  = 7;
    // 本轮游戏参与地址数
    int64 userCount = 8;
    // 本轮游戏募集到的key个数
    int64 keyCount = 9;
    //距离开奖剩余时间
    int64 remainTime = 10;
}
```

**每笔交易中的Key信息：**

```protobuf
message KeyInfo {
    // 游戏轮次  (是由系统合约填写后存储）
    int64 round = 1;
      
    // 本次购买key的价格 (是由系统合约填写后存储）
    int64 keyPrice = 2;  
    
    // 用户本次买的key的数量
    int64 keyNum = 3;
    
    // 用户地址 (是由系统合约填写后存储）
    string addr = 4;
    
    // 本次买key总共花费多少
    int64 totalCost = 5
    
    // 交易确认存储时间（被打包的时间）
    int64 buyKeyTime = 6; 
    
    //买票的txHash
    string buyKeyTxHash = 7;
    
}
```

### 1.1.2. 链上存储 StateDB
|键|值|用途|说明|
|-|-|-|-|
|mavl-f3d-user-keys:{round}:{addr}|钥匙把数|某一轮游戏中用户持有的钥匙数|记录游戏参与者持有的钥匙数目，变参为游戏轮次和用户地址|
|mavl-f3d-round-start|轮次开始|存储开始的轮次信息|
|mavl-f3d-round-end|轮次结束|存储结束的轮次信息|
|mavl-f3d-last-round|当前轮次|保存当前轮次||
|mavl-f3d-key-price:{round}|钥匙价格|存储最后一把钥匙的价格|
|mavl-f3d-buy-key:{round}:{addr}:{index}|key信息|存储每笔交易key的信息|

### 1.1.3. 本地存储 LocalDB
|键|值|用途|说明|
|-|-|-|-|
|LODB-f3d-round-info:{round}|RoundInfo|保存每一个轮次的游戏信息||

用户买票存储

|键|值|用途|说明|
|-|-|-|-|
|LODB-f3d-buy:{round}:{addr}:{index}|index,addr信息|记录用户每次买key存储在stateDB上的，key中的index|变参为游戏轮次，用户地址，index|


**问题点**
1. 如果每笔买入key的交易信息都上链，存储在stateDB中，做到可溯源的话，存储的数据会比较多

  ~~2. 当开奖交易触发时，需要更改stateDB中之前每笔买Key的bonus等字段信息，需要通过localDB list 辅助去查询更改，极端情况下，如果买的人很多的话，
  为了系统的安全，需要分多次去查，然后分批处理，这样会不会影响性能？ 可以考虑把这个bonus去掉，这样不用二次写入~~
  
  
  3.不用去二次更改，只存一次


## 1.2. 接口设计



**tx中构造action设计：**
```protobuf
// message for execs.f3d
message F3dAction {
    oneof value {
        F3dStart start = 1;
        F3dLuckyDraw draw = 2;
        F3dBuyKey  buy  = 3;
    }
    int32 ty = 4;
}

message F3dStart {
    //轮次，这个填不填不重要，合约里面会自动校验的
    int64 Round  = 1;
}

message F3dLuckyDraw {
    //轮次，这个填不填不重要，合约里面会自动校验的
    int64 Round  = 1;
}

message  {
    // 用户本次买的key的数量
    int64 keyNum = 3;
}
```


**对外查询接口设计：**

f3d round 信息查询
``` protobuf
// 查询f3d 游戏信息,这里面其实包含了key的最新价格信息
message QueryF3dByRound {
    //轮次，默认查询最新的
    int64 round = 1;
}

message QueryF3dListByRound {
    //轮次，默认查询最新的
    int64 startRound = 1;
    //单页返回多少条记录，默认返回10条，单次最多返回50条
    int32 count = 2;
    // 0降序，1升序，默认降序
    int32 direction = 5;
}
```

**买的钥匙key信息记录查询**

```protobuf
// key 信息查询
message QueryKeysByRoundAndAddr {
    //轮次,必填参数
    int64 round = 1;
    //用户地址
    string addr = 2;
}
// 用户key数量查询
message QueryKeyCountByRoundAndAddr {
    //轮次,必填参数
    int64 round = 1;
    //用户地址
    string addr = 2;
}
```

**内部索引value值**

``` protobuf
message F3dRecord {
    //用户地址
    string addr = 1;
    //index
    int64  index  = 2;
    //round
    int64  round = 3;
}
```

**响应数据**
``` protobuf
//f3d round查询返回数据 
message ReplyF3dList {
    repeated RoundInfo rounds = 1;
}

message ReplyF3d {
    RoundInfo round = 1;
}

//用户查询买的key信息返回数据
message ReplyKeyList {
    repeated KeyInfo keys = 1;
}

message ReplyKey {
    KeyInfo key = 1;
}

message ReplyKeyCount {
    int64 count = 1;
}

//合约内部日志记录，待补全
message ReceiptF3d {
    string addr = 1;
    int64  round = 2;
    int64  index      = 3;
   .....
}
```







### 1.2.1. 全局接口
#### 1.2.1.1. 获取游戏轮次信息 GetGameRounds
**请求结构：**
```json
{
    "startRound":int64,
    "endRound":int64
}
```

|参数|类型|是否必填|说明|
|----|----|----|----|----|
|startRound|int64|是|游戏开始轮次，<=0时表示当前轮次，否则为指定轮次|
|endRound|int64|是|游戏结束轮次，<=0时表示当前轮次，否则为指定轮次|

**返回结构：**

返回RoundInfo数组的json结构信息

#### 1.2.1.2 开奖 EndGameRound
使用CreateTransaction接口

**请求结构：**
```json
{
    "round":int64
}
```

|参数|类型|是否必填|说明|
|----|----|----|----|----|
|round|int64|是|游戏轮次|

**返回结构：**

RoundInfo

#### 1.2.1.3 开始下一轮游戏 StartGameRound
使用CreateTransaction接口

**请求结构：**
```json
{
    "lastRound":int64
}
```

|参数|类型|是否必填|说明|
|----|----|----|----|----|
|lastRound|int64|是|上一轮游戏轮次，如果<=0，则认为启动第一轮|

**返回结构：**

RoundInfo

### 1.2.2. 用户接口
#### 1.2.2.1. 创建购买钥匙交易
使用CreateTransaction接口

**请求结构：**
```json
{
    "keyNum":int64
}
```

|参数|类型|是否必填|说明|
|----|----|----|----|----|
|keyNum|int64|是|购买钥匙把数，>0有效|

#### 1.2.2.2. 获取用户的中奖信息 GetUserBonus
**请求结构：**
```json
{
    "user":"string",
    "startRound":int64,
    "endRound":int64
}
```

|参数|类型|是否必填|说明|
|----|----|----|----|----|
|user|string|是|游戏参与者的地址|
|startRound|int64|是|开始轮次|
|endRound|int64|是|结束轮次，<=0时，表示当前轮次|

> 数据查询范围为闭区间 [startRound, endRound]

**返回结构：**
```json
{
    [
        {
            "round":int64,
            "keyNum":int64,
            "bonus":int64
        }
    ]
}
```

|参数|类型|说明|
|----|----|----|
|round|int64|游戏轮次|
|keyNum|int64|持有的钥匙把数|
|bonus|int64|拿到的奖金|


## 1.3. 缓存设计
暂无，待需要时补充

## 1.4. 合约配置

合约配置信息，支持以下配置项：
```ini
# 本游戏合约管理员地址
f3d.manager = 14KEKbYtKKQm4wMthSK9J4La4nAiidGozt

# 本游戏合约平台开发者分成地址
f3d.developer = 12qyocayNF7Lv6C9qW4avxs2E7U41fKSfv

# 超级大奖分成百分比
f3d.bonus.winner = 40

# 参与者分红百分比
f3d.bonus.key = 30

# 滚动到下期奖金池百分比
f3d.bonus.pool = 20

# 平台运营及开发者费用百分比
f3d.bonus.developer = 10

# 本游戏一轮运行的最长周期（单位：秒）
f3d.time.life = 3600

# 一把钥匙延长的游戏时间（单位：秒）
f3d.time.key = 30

# 一次购买钥匙最多延长的游戏时间（单位：秒）
f3d.time.maxkey = 300

# 钥匙涨价幅度（下一个人购买钥匙时在上一把钥匙基础上浮动幅度百分比），范围1-100
f3d.key.price.incr=10
```

