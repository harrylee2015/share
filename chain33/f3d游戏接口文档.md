# f3d接口文档

# 一、接口描述 
## 1 开启F3D游戏
### 1.1 生成开启F3D游戏的交易（未签名）
**请求报文：**
```json
  request: http.post
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "Chain33.CreateTransaction",
  "params": [
    {
      "execer": "f3d",
      "actionName": "Start",
    }
  ]
}
```
**参数说明：**

|参数|类型|说明|
|----|----|----|
|execer|string|固定f3d合约名称
|actionName|string|这里还是固定值Start|

**响应报文：**
```json
response:
{
  "id": 2,
  "result": "0a12757365722e702e67756573732e6775657373128c0138050a87010a0e576f726c644375702046696e616c1212413a4672616e63653b423a436c616f6469611a08666f6f7462616c6c2880c8afa0253080d0dbc3f4023805422231443652465a4e7032726836516462635a31643752577542557a3631576536534437480552223150487443684e743355636673735237763774724b536b33574a7441576a4b6a6a5820c0843d30cbbffaeda9b1dcd2023a223139455a316e677138703254507335544c70455a77546a6e6e416766746159616e68",
  "error": null
}
```
**参数说明：**

|参数|类型|说明|
|----|----|----|
|result|string|交易十六进制编码后的字符串|


#### 1.2 交易签名 SignRawTx

**请求报文：**
```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "Chain33.SignRawTx",
  "params": [
    {
      "addr": "14KEKbYtKKQm4wMthSK9J4La4nAiidGozt",
      "privKey": "0x8d8523fbd09c27dabc853e99daf167533acaa1e2a2498dc2cb7449cff8137a37",
      "txHex": "0a12757365722e702e67756573732e6775657373128c0138050a87010a0e576f726c644375702046696e616c1212413a4672616e63653b423a436c616f6469611a08666f6f7462616c6c2880c8afa0253080d0dbc3f4023805422231443652465a4e7032726836516462635a31643752577542557a3631576536534437480552223150487443684e743355636673735237763774724b536b33574a7441576a4b6a6a5820a08d0630fe86c8ccf4b490a0323a223139455a316e677138703254507335544c70455a77546a6e6e416766746159616e68",
      "expire": "3600s",
      "index": 0,
      "model": 0
    }
  ]
}
```

**参数说明：**

|参数|类型|是否必填|说明|
|----|----|----|----|
|addr|string|否|addr与privkey可以只输入其一，如果使用addr则依赖钱包中存储的私钥签名|
|privkey|string|否|addr与privkey可以只输入其一，如果使用privkey则直接签名|
|txHex|string|是|上一步生成的原始交易数据|
|expire|string|是|过期时间可输入如"300ms"，"-1.5h"或者"2h45m"的字符串，有效时间单位为"ns", "us" (or "µs"), "ms", "s", "m", "h"|
|index|int32|否|若是签名交易组，则为要签名的交易序号，从1开始，小于等于0则为签名组内全部交易|
|token|string|否||

**响应报文：**
```json
{
  "id": 1,
  "result": "0a12757365722e702e67756573732e6775657373128c0138050a87010a0e576f726c644375702046696e616c1212413a4672616e63653b423a436c616f6469611a08666f6f7462616c6c2880c8afa0253080d0dbc3f4023805422231443652465a4e7032726836516462635a31643752577542557a3631576536534437480552223150487443684e743355636673735237763774724b536b33574a7441576a4b6a6a581a6e0801122102504fa1c28caaf1d5a20fefb87c50a49724ff401043420cb3ba271997eb5a43871a473045022100ad8dcafbefbea043e861cced9cca766ec724506821df33b32f49559aea84a34d022035be794beb88186bf5023e589babde1a6c5a59bb248248f3e3669bfd2f9a48ad20a08d0628fda2d1e10530fe86c8ccf4b490a0323a223139455a316e677138703254507335544c70455a77546a6e6e416766746159616e68",
  "error": null
}
```

**参数说明：**

|参数|类型|说明|
|----|----|----|
|result|string|交易签名后的十六进制字符串|


#### 1.3 发送交易 SendTransaction
**请求报文：**
```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "Chain33.SendTransaction",
  "params": [
    {
      "data": "0a12757365722e702e67756573732e6775657373128c0138050a87010a0e576f726c644375702046696e616c1212413a4672616e63653b423a436c616f6469611a08666f6f7462616c6c2880c8afa0253080d0dbc3f4023805422231443652465a4e7032726836516462635a31643752577542557a3631576536534437480552223150487443684e743355636673735237763774724b536b33574a7441576a4b6a6a581a6e0801122102504fa1c28caaf1d5a20fefb87c50a49724ff401043420cb3ba271997eb5a43871a473045022100ad8dcafbefbea043e861cced9cca766ec724506821df33b32f49559aea84a34d022035be794beb88186bf5023e589babde1a6c5a59bb248248f3e3669bfd2f9a48ad20a08d0628fda2d1e10530fe86c8ccf4b490a0323a223139455a316e677138703254507335544c70455a77546a6e6e416766746159616e68"
    }
  ]
}
```

**参数说明：**

|参数|类型|是否必填|说明|
|----|----|----|----|
|data|string|是|为上一步签名后的交易数据|

**响应报文：**
```json
{
  "id": 2,
  "result": "0xd9dddff84753e18f90231ea2c928df6ef25eef24a051f34d22d262b8d3dc92b4",
  "error": null
}
```
**参数说明：**

|参数|类型|说明|
|----|----|----|
|result|string|交易发送后，生成的交易哈希（后面可以使用此哈希查询交易状态和历史）|

## 2 参与f3d游戏
### 2.1 生成买key的交易（未签名）
**请求报文：**
```json
  request: http.post
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "Chain33.CreateTransaction",
  "params": [
    {
      "execer": "f3d",
      "actionName": "Buy",
      "payload": {
        "keyNum": 100
      }
    }
  ]
}
```
**参数说明：**

|参数|类型|说明|
|----|----|----|
|keyNum|int64|买多少张票|

**响应报文：**
```json
response:
{
  "id": 2,
  "result": "0a12757365722e702e67756573732e677565737312513806124d0a423078643964646466663834373533653138663930323331656132633932386466366566323565656632346130353166333464323264323632623864336463393262341201411880cab5ee0120a08d06308e9087f9a4f9effd393a223139455a316e677138703254507335544c70455a77546a6e6e416766746159616e68",
  "error": null
}
```
**参数说明：**

|参数|类型|说明|
|----|----|----|
|result|string|交易十六进制编码后的字符串|

## ３ 停止并开奖某一轮f3d
### 3.1 生成某局游戏开奖的交易（未签名）
**请求报文：**
```json
  request: http.post
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "Chain33.CreateTransaction",
  "params": [
    {
      "execer": "f3d",
      "actionName": "Draw",
    }
  ]
}
```

说明：　　
该交易必须由游戏发起地址来创建。

**响应报文：**
```json
response:
{
  "id": 2,
  "result": "0a12757365722e702e67756573732e6775657373124838071a440a4230786439646464666638343735336531386639303233316561326339323864663665663235656566323461303531663334643232643236326238643364633932623420a08d0630a58a95b2fbd791de2d3a223139455a316e677138703254507335544c70455a77546a6e6e416766746159616e68",
  "error": null
}
```
**参数说明：**

|参数|类型|说明|
|----|----|----|
|result|string|交易十六进制编码后的字符串|

## 3 查询接口
### 3.1 查询最新轮次的f3d游戏信息
**请求报文：**

```json
request: http.post
{
  "jsonrpc": "2.0",
  "id": 0,
  "method": "Chain33.Query",
  "params": [
    {
      "execer": "f3d",
      "funcName": "QueryLastRoundInfo",
      "payload": {
      }
    }
  ]
}
```

**参数说明：**

|参数|类型|说明|
|----|----|----|
|method|string|Chain33.Query|
|execer|string|执行器名，主链上:f3d, 平行链上:user.p.xxx.f3d|
|funcName|string|QueryLastRoundInfo|


**响应报文：**

```json
response:
{
    "id":1,
    "result":   {
             "round":"3",
             "beginTime":"1544597303",
             "endTime":"0",
             "lastOwner":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh",
             "lastKeyTime":"1544597710",
             "lastKeyPrice":0.121,
             "bonusPool":21,
             "userCount":"1",
             "keyCount":"200",
             "remainTime":"3600",
             "updateTime":"1544597710"
              },
     "error":null
 }
```

**参数说明：**

|参数|类型|说明|
|----|----|----|
|round|int64|第几轮|
|beginTime|int64|开始时间|
|endTime|int64|结束时间|
|lastOwner|string|最后一把钥匙持有人地址|
|lastKeyTime|int64|最后一把钥匙买入时间|
|lastKeyPrice|int64|最后一把钥匙的价格|
|bonusPool|float|奖池累计金额|
|userCount|int64|累计有多少用户参与这一轮游戏|
|keyCount|int64|已售出key的总数|
|remainTime|int64|游戏剩余时间/s|
|updateTime|int64|上一次更新时间|

### 3.2 查询指定轮次的f3d游戏信息

**请求报文：**

```json
request: http.post
{
  "jsonrpc": "2.0",
  "id": 0,
  "method": "Chain33.Query",
  "params": [
    {
      "execer": "f3d",
      "funcName": "QueryLastRoundInfo",
      "payload": {
      }
    }
  ]
}
```

**参数说明：**

|参数|类型|说明|
|----|----|----|
|method|string|Chain33.Query|
|execer|string|执行器名，主链上:f3d, 平行链上:user.p.xxx.f3d|
|funcName|string|QueryLastRoundInfo|


**响应报文：**

```json
response:
{
    "id":1,
    "result":   {
             "round":"3",
             "beginTime":"1544597303",
             "endTime":"0",
             "lastOwner":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh",
             "lastKeyTime":"1544597710",
             "lastKeyPrice":0.121,
             "bonusPool":21,
             "userCount":"1",
             "keyCount":"200",
             "remainTime":"3600",
             "updateTime":"1544597710"
              },
     "error":null
 }
```

**参数说明：**

|参数|类型|说明|
|----|----|----|
|round|int64|第几轮|
|beginTime|int64|开始时间|
|endTime|int64|结束时间|
|lastOwner|string|最后一把钥匙持有人地址|
|lastKeyTime|int64|最后一把钥匙买入时间|
|lastKeyPrice|int64|最后一把钥匙的价格|
|bonusPool|float|奖池累计金额|
|userCount|int64|累计有多少用户参与这一轮游戏|
|keyCount|int64|已售出key的总数|
|remainTime|int64|游戏剩余时间/s|
|updateTime|int64|上一次更新时间|

### 3.3 根据地址和轮次查询用户收益情况

**请求报文：**

```json
request: http.post
{
  "jsonrpc": "2.0",
  "id": 0,
  "method": "Chain33.Query",
  "params": [
    {
      "execer": "f3d",
      "funcName": "QueryLastRoundInfo",
      "payload": {
          "round":3,
          "addr":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
      }
    }
  ]
}
```

**参数说明：**

|参数|类型|说明|
|----|----|----|
|method|string|Chain33.Query|
|execer|string|执行器名，主链上:f3d, 平行链上:user.p.xxx.f3d|
|funcName|string|QueryKeyCountByRoundAndAddr|
|round|int64|第几轮|
|addr|string|用户地址|


**响应报文：**

```json

response:

{ 
   "id":1,
   "result":
        {
        "addr":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh",
        "keyNum":"200",
        "buyCount":"2",
        "round":"3"，
        "bonus":0,
        "totalCost":21
        },
    "error":null
 }
```

**参数说明：**

|参数|类型|说明|
|----|----|----|
|round|int64|第几轮|
|keyNum|int64|本轮游戏key买的总数|
|buyCount|int64|本轮游戏买了几次|
|addr|string|用户地址|
|bonus|int64|奖金，没开奖时，默认是0|
|totalCost|int64|买钥匙总成本|


### 3.3 查询用户具体的买票信息
