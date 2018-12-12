## F3D游戏玩法测试

### 1.准备工作分别给manage地址和测试地址打钱，并奖 测试地址中的一部分钱转到f3d合约下面

``` bash
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli bty transfer -a 10000 -t 16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh
0a05636f696e73122f18010a2b1080a094a58d1d2222313652655a487a4d434774507438423758626e5a51326a655873504739774575666820a08d0630f9b8cadba0a0f1ac083a22313652655a487a4d434774507438423758626e5a51326a6558735047397745756668
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli wallet sign -k "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -d "0a05636f696e73122f18010a2b1080a094a58d1d2222313652655a487a4d434774507438423758626e5a51326a655873504739774575666820a08d0630f9b8cadba0a0f1ac083a22313652655a487a4d434774507438423758626e5a51326a6558735047397745756668"
0a05636f696e73122f18010a2b1080a094a58d1d2222313652655a487a4d434774507438423758626e5a51326a65587350473977457566681a6d0801122102504fa1c28caaf1d5a20fefb87c50a49724ff401043420cb3ba271997eb5a43871a46304402200f45957571161769a0ce3a15de0ea290db87f8c823a828d7fa55414fab5c7f0e02204383c95c400890e98355563281cf4ce66ecbe1fcf0a1eb51028adc4a4aea2fbe20a08d0628c9fcc1e00530f9b8cadba0a0f1ac083a22313652655a487a4d434774507438423758626e5a51326a6558735047397745756668
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli wallet send -d "0a05636f696e73122f18010a2b1080a094a58d1d2222313652655a487a4d434774507438423758626e5a51326a65587350473977457566681a6d0801122102504fa1c28caaf1d5a20fefb87c50a49724ff401043420cb3ba271997eb5a43871a46304402200f45957571161769a0ce3a15de0ea290db87f8c823a828d7fa55414fab5c7f0e02204383c95c400890e98355563281cf4ce66ecbe1fcf0a1eb51028adc4a4aea2fbe20a08d0628c9fcc1e00530f9b8cadba0a0f1ac083a22313652655a487a4d434774507438423758626e5a51326a6558735047397745756668"
0xd45753d3f5427813abf15f6a779dab676f8e96ad2e7dd77200f593f69f1e5e3b
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli account list
{
    "wallets": [
        {
            "acc": {
                "balance": "0.0000",
                "frozen": "0.0000",
                "addr": "17Dgq9NQdTfjvu18uLMmHshA5mDzjum4XD"
            },
            "label": "node award"
        },
        {
            "acc": {
                "balance": "14606242.1160",
                "frozen": "0.0000",
                "addr": "14KEKbYtKKQm4wMthSK9J4La4nAiidGozt"
            },
            "label": "genGame"
        },
        {
            "acc": {
                "balance": "10000.0000",
                "frozen": "0.0000",
                "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
            },
            "label": "tokenOWN"
        }
    ]
}
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli bty send_exec -a 1000  -e f3d
0a05636f696e731234180a2a301080d0dbc3f40222036633642a223141754e316a58417537666a69666a4c6d6771385732366a325241636f714667557620a08d0630ca92ffabbfb2f7f6573a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576
#签名,xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx，是自己的私钥
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli wallet sign -k "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" -d "0a05636f696e731234180a2a301080d0dbc3f40222036633642a223141754e316a58417537666a69666a4c6d6771385732366a325241636f714667557620a08d0630ca92ffabbfb2f7f6573a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576"
0a05636f696e731234180a2a301080d0dbc3f40222036633642a223141754e316a58417537666a69666a4c6d6771385732366a325241636f71466755761a6e08011221021832ee50ab60016d6f6bf03315b9354434ba487929c57c79d67de2309aba3bfe1a47304502210094b5d740e13fa9d6c0b6740750ffce08a9e71ab62b4b6a0fc8a5bb02eae93c6f02207ae01f335b1f68a2b94f94f60f2300fa2dd08945f265853d4cf16df8edc5ceda20a08d0628aefdc1e00530ca92ffabbfb2f7f6573a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli wallet send -d "0a05636f696e731234180a2a301080d0dbc3f40222036633642a223141754e316a58417537666a69666a4c6d6771385732366a325241636f71466755761a6e08011221021832ee50ab60016d6f6bf03315b9354434ba487929c57c79d67de2309aba3bfe1a47304502210094b5d740e13fa9d6c0b6740750ffce08a9e71ab62b4b6a0fc8a5bb02eae93c6f02207ae01f335b1f68a2b94f94f60f2300fa2dd08945f265853d4cf16df8edc5ceda20a08d0628aefdc1e00530ca92ffabbfb2f7f6573a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576"
0xfa13f63d5126b2ed377d4011782e805792fec727e4c735582b08701e8b09a613
```

### 2.启动游戏
``` bash
harry@harry-VirtualBox:~/chain33Test$  curl --data-binary '{"jsonrpc":"2.0", "id": 1, "method":"Chain33.CreateTransaction","params":[{"execer":"f3d", "actionName":"Start", "payload":{"Round":0}}]} ' \
>         -H 'content-type:text/plain;' \
>         http://localhost:8801
{"id":1,"result":"0a03663364120420010a0020a08d0630a8a9cfc9f397fcbe573a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576","error":null}

# 同上用管理地址去签名，然后发送
```

## 3.其他玩家买票
``` bash
harry@harry-VirtualBox:~/chain33Test$ curl --data-binary '{"jsonrpc":"2.0", "id": 1, "method":"Chain33.CreateTransaction","params":[{"execer":"f3d", "actionName":"Buy", "payload":{"keyNum":100}}]} '         -H 'content-type:text/plain;'         http://localhost:8801  
{"id":1,"result":"0a03663364120620031a02186420a08d0630da9cce81d5ddcf87283a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576","error":null}

#签名 略
#发送
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli wallet send -d "0a03663364120620031a0218641a6e08011221021832ee50ab60016d6f6bf03315b9354434ba487929c57c79d67de2309aba3bfe1a473045022100d7ad3f9d88a0731b677b1647480e3e851ad0d75edfdba188bac6a8be653b021f022030b94d5980bfd36832a4411b6faf30fe7c7fbbd18a714224c56d3b0dd77e5d8420a08d0628e5dfc2e00530da9cce81d5ddcf87283a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576"
0xb23d5e37ae54fd2ca9fc981ba2737c430dc9c5b06eaa5be68dd7eeae0751c6aa
#查看执行结果
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli  tx query -s 0xb23d5e37ae54fd2ca9fc981ba2737c430dc9c5b06eaa5be68dd7eeae0751c6aa
{
    "tx": {
        "execer": "f3d",
        "payload": {
            "buy": {
                "keyNum": "100"
            },
            "ty": 3
        },
        "rawpayload": "0x20031a021864",
        "signature": {
            "ty": 1,
            "pubkey": "0x021832ee50ab60016d6f6bf03315b9354434ba487929c57c79d67de2309aba3bfe",
            "signature": "0x3045022100d7ad3f9d88a0731b677b1647480e3e851ad0d75edfdba188bac6a8be653b021f022030b94d5980bfd36832a4411b6faf30fe7c7fbbd18a714224c56d3b0dd77e5d84"
        },
        "fee": "0.0010",
        "expire": 1544597477,
        "nonce": 2886595075141504602,
        "to": "1AuN1jXAu7fjifjLmgq8W26j2RAcoqFgUv",
        "from": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh",
        "hash": "0xb23d5e37ae54fd2ca9fc981ba2737c430dc9c5b06eaa5be68dd7eeae0751c6aa"
    },
    "receipt": {
        "ty": 2,
        "tyName": "ExecOk",
        "logs": [
            {
                "ty": 2,
                "tyName": "LogFee",
                "log": {
                    "prev": {
                        "currency": 0,
                        "balance": "49999700000",
                        "frozen": "0",
                        "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
                    },
                    "current": {
                        "currency": 0,
                        "balance": "49999600000",
                        "frozen": "0",
                        "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
                    }
                },
                "rawLog": "0x0a2b10a0c0dba1ba012222313652655a487a4d434774507438423758626e5a51326a6558735047397745756668122b1080b3d5a1ba012222313652655a487a4d434774507438423758626e5a51326a6558735047397745756668"
            },
            {
                "ty": 6,
                "tyName": "LogExecTransfer",
                "log": {
                    "execAddr": "1AuN1jXAu7fjifjLmgq8W26j2RAcoqFgUv",
                    "prev": {
                        "currency": 0,
                        "balance": "50000000000",
                        "frozen": "0",
                        "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
                    },
                    "current": {
                        "currency": 0,
                        "balance": "49000000000",
                        "frozen": "0",
                        "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
                    }
                },
                "rawLog": "0x0a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576122b1080e8eda1ba012222313652655a487a4d434774507438423758626e5a51326a65587350473977457566681a2b1080d482c5b6012222313652655a487a4d434774507438423758626e5a51326a6558735047397745756668"
            },
            {
                "ty": 6,
                "tyName": "LogExecTransfer",
                "log": {
                    "execAddr": "1AuN1jXAu7fjifjLmgq8W26j2RAcoqFgUv",
                    "prev": {
                        "currency": 0,
                        "balance": "0",
                        "frozen": "0",
                        "addr": "14KEKbYtKKQm4wMthSK9J4La4nAiidGozt"
                    },
                    "current": {
                        "currency": 0,
                        "balance": "1000000000",
                        "frozen": "0",
                        "addr": "14KEKbYtKKQm4wMthSK9J4La4nAiidGozt"
                    }
                },
                "rawLog": "0x0a223141754e316a58417537666a69666a4c6d6771385732366a325241636f71466755761224222231344b454b6259744b4b516d34774d7468534b394a344c61346e41696964476f7a741a2a108094ebdc03222231344b454b6259744b4b516d34774d7468534b394a344c61346e41696964476f7a74"
            },
            {
                "ty": 9,
                "tyName": "LogExecFrozen",
                "log": {
                    "execAddr": "1AuN1jXAu7fjifjLmgq8W26j2RAcoqFgUv",
                    "prev": {
                        "currency": 0,
                        "balance": "1000000000",
                        "frozen": "0",
                        "addr": "14KEKbYtKKQm4wMthSK9J4La4nAiidGozt"
                    },
                    "current": {
                        "currency": 0,
                        "balance": "0",
                        "frozen": "1000000000",
                        "addr": "14KEKbYtKKQm4wMthSK9J4La4nAiidGozt"
                    }
                },
                "rawLog": "0x0a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576122a108094ebdc03222231344b454b6259744b4b516d34774d7468534b394a344c61346e41696964476f7a741a2a188094ebdc03222231344b454b6259744b4b516d34774d7468534b394a344c61346e41696964476f7a74"
            },
            {
                "ty": 103,
                "tyName": "LogBuyF3d",
                "log": {
                    "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh",
                    "round": "3",
                    "index": "84700001",
                    "action": "3",
                    "buyCount": "1",
                    "keyNum": "0"
                },
                "rawLog": "0x0a22313652655a487a4d434774507438423758626e5a51326a6558735047397745756668100318e1d6b12820032801"
            }
        ]
    },
    "proofs": [
        "0x5ca83d0e3701fa39bbd3b8aa2309e17cfbeee819782b77d5d990b0d8f949ce0b"
    ],
    "height": 847,
    "index": 1,
    "blocktime": 1544597393,
    "amount": "0.0000",
    "fromaddr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh",
    "actionname": "buy",
    "assets": null
}

```

### 4.F3d 对提供的查询接口

#### 4.1 查询最新的round信息

``` bash
harry@harry-VirtualBox:~/chain33Test$  curl --data-binary '{"jsonrpc":"2.0", "id": 1, "method":"Chain33.Query","params":[{"execer":"f3d", "funcName":"QueryLastRoundInfo", "payload":{}}]} ' \
>         -H 'content-type:text/plain;' \
>         http://localhost:8801
{"id":1,"result":{"round":"3","beginTime":"1544597303","endTime":"0","lastOwner":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh","lastKeyTime":"1544597710","lastKeyPrice":0.121,"bonusPool":21,"userCount":"1","keyCount":"200","remainTime":"3600","updateTime":"1544597710"},"error":null}

```

#### 4.2 查询指定round的f3d信息

``` bash
harry@harry-VirtualBox:~/chain33Test$ curl --data-binary '{"jsonrpc":"2.0", "id": 1, "method":"Chain33.Query","params":[{"execer":"f3d", "funcName":"QueryRoundInfoByRound", "payload":{"round":3}}]} ' \
>         -H 'content-type:text/plain;' \
>         http://localhost:8801
{"id":1,"result":{"round":"3","beginTime":"1544597303","endTime":"0","lastOwner":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh","lastKeyTime":"1544597710","lastKeyPrice":0.121,"bonusPool":21,"userCount":"1","keyCount":"200","remainTime":"3600","updateTime":"1544597710"},"error":null}

```

#### 4.3 查询指定地址的买票数

``` bash
harry@harry-VirtualBox:~/chain33Test$  curl --data-binary '{"jsonrpc":"2.0", "id": 1, "method":"Chain33.Query","params":[{"execer":"f3d", "funcName":"QueryKeyCountByRoundAndAddr", "payload":{"round":3,"addr":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"}}]} ' \
>         -H 'content-type:text/plain;' \
>         http://localhost:8801
{"id":1,"result":{"addr":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh","keyNum":"200","buyCount":"2","round":"3"},"error":null}

```
#### 4.4 查询购票记录

``` bash
harry@harry-VirtualBox:~/chain33Test$  curl --data-binary '{"jsonrpc":"2.0", "id": 1, "method":"Chain33.Query","params":[{"execer":"f3d", "funcName":"QueryBuyRecordByRoundAndAddr", "payload":{"round":3,"addr":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"}}]} ' \
>         -H 'content-type:text/plain;' \
>         http://localhost:8801
{"id":1,"result":{"recordList":[{"round":"3","keyPrice":0.11,"keyNum":"100","addr":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh","buyKeyTime":"1544597710","buyKeyTxHash":"0x5dabfce7808cade711b463c77333c24237546525cf460eb66d9036dfd7d15739","index":"86800001"},{"round":"3","keyPrice":0.1,"keyNum":"100","addr":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh","buyKeyTime":"1544597393","buyKeyTxHash":"0xb23d5e37ae54fd2ca9fc981ba2737c430dc9c5b06eaa5be68dd7eeae0751c6aa","index":"84700001"}]},"error":null}

```

#### 5. 开奖

``` bash
harry@harry-VirtualBox:~/chain33Test$ curl --data-binary '{"jsonrpc":"2.0", "id": 1, "method":"Chain33.CreateTransaction","params":[{"execer":"f3d", "actionName":"Draw", "payload":{"Round":3}}]} ' \
>         -H 'content-type:text/plain;' \
>         http://localhost:8801
{"id":1,"result":"0a03663364120620021202080320a08d063086e4cfbfede99887073a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576","error":null}
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli wallet sign -k "cc38546e9e659d15e6b4893f0ab32a06d103931a8230b0bde71459d2b27d6944" -d "0a03663364120620021202080320a08d063086e4cfbfede99887073a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576"
0a0366336412062002120208031a6d0801122102504fa1c28caaf1d5a20fefb87c50a49724ff401043420cb3ba271997eb5a43871a46304402201b32ec4a8d861a7c53f701b0c7cd580924c00edbdf73f2f6a0c8f77113bf50ce02202dae8655e2d284a9429981996f85df339890704a597176875ffd505919c9b1a720a08d0628df8ac3e0053086e4cfbfede99887073a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli wallet send -d "0a0366336412062002120208031a6d0801122102504fa1c28caaf1d5a20fefb87c50a49724ff401043420cb3ba271997eb5a43871a46304402201b32ec4a8d861a7c53f701b0c7cd580924c00edbdf73f2f6a0c8f77113bf50ce02202dae8655e2d284a9429981996f85df339890704a597176875ffd505919c9b1a720a08d0628df8ac3e0053086e4cfbfede99887073a223141754e316a58417537666a69666a4c6d6771385732366a325241636f7146675576"
0x51d9b7d323490636a3cde21d2bd155ea5c67ae718c602b329b26b3528a7a4257
```

#### 6. 查看地址余额信息

  ........
