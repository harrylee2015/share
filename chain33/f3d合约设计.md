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

### 1.1.3. 本地存储 LocalDB
|键|值|用途|说明|
|-|-|-|-|
|LODB-f3d-round-info:{round}|RoundInfo|保存每一个轮次的游戏信息||

用户买票存储

|键|值|用途|说明|
|-|-|-|-|
|LODB-f3d-buy:{round}|{addr}|游戏轮数|用户地址|




## 1.2. 接口设计
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
