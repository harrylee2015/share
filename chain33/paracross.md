# 跨链币使用

目前chain33 支持把资产 从主链跨链到平行链使用。 具体步骤如下

 1. 在主链上先把资产转移到paracross 合约。 目前资产有 coins 和 token 的币
 
 ```bash
 harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli  send  bty  send_exec  -a  100  -e  paracross -k "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
 ```
 * 结果
 ```bash
 harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli account balance -a "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh" -e  paracross
{
    "balance": "100.0000",
    "frozen": "0.0000",
    "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
}

 ```
 2. 调用 paracross 合约的 asset_transfer 交易， 将币转移到 指定的平行链
    对应的币就会出现在 对应的平行链的 paracross 合约
```bash
harry@harry-VirtualBox:~/chain33Test$ ./chain33-cli send  para asset_transfer --title user.p.evmtest. -a 2 -n test -t "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh" -k "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
0x55b4aa75a7121d2a3da0ba66f229d364ac95c45251e7218b58317d061542ecd5
```
* 结果
下面查询方法有误，暂时不支持查询（paracross币是特有的币，和coins不一样，暂时查询接口还没开发，这个和林敬确认过）
```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli asset balance --addr "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh" --asset_exec paracross coins  --asset_symbol coins.bty  -e  "paracross"   --rpc_laddr http://localhost:8901
{
    "balance": "6.0000",
    "frozen": "0.0000",
    "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
}
```

 3. 调用平行链的 paracross 合约的 send_exec 将 币 转移到 trade 合约
 * 查询合约地址
```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli  exec addr -e  trade  --rpc_laddr http://localhost:8901  --paraName user.p.evmtest.
1BXvgjmBw1aBgmGn1hjfGyRkmN3krWpFP4
```

* 转币

```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli send   para transfer_exec -a 1 -n test -s coins.bty --title user.p.evmtest. -t 1BXvgjmBw1aBgmGn1hjfGyRkmN3krWpFP4  -k "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh" --rpc_laddr http://localhost:8901  --paraName user.p.evmtest.
0x3a2b0e21a806c32d79c14d34dd7ee467eb1c4be05fb3dc1b1a58c7b3e3c1a44a
```
* 结果余额不足，这个需要重新确认
```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli tx query_hash -s 0x3a2b0e21a806c32d79c14d34dd7ee467eb1c4be05fb3dc1b1a58c7b3e3c1a44a  --rpc_laddr http://localhost:8901  --paraName user.p.evmtest.
{
    "txs": [
        {
            "tx": {
                "execer": "user.p.evmtest.paracross",
                "payload": {
                    "transfer": {
                        "cointoken": "coins.bty",
                        "amount": "100000000",
                        "note": "0x74657374",
                        "to": "1BXvgjmBw1aBgmGn1hjfGyRkmN3krWpFP4"
                    },
                    "ty": 2
                },
                "rawpayload": "0x1002323a0a09636f696e732e6274791080c2d72f1a0474657374222231425876676a6d4277316142676d476e31686a664779526b6d4e336b725770465034",
                "signature": {
                    "ty": 1,
                    "pubkey": "0x021832ee50ab60016d6f6bf03315b9354434ba487929c57c79d67de2309aba3bfe",
                    "signature": "0x304402207f56bdaa891f13eb8380245d75e56567d51fbca6add9d6390579ab1ba304744502206249fa21e9fe5307ff1e465a2b0e18abe2feb6d6aa779489aec8f26eabb80841"
                },
                "fee": "0.0010",
                "expire": 1546070739,
                "nonce": 8166473135691879338,
                "to": "1BXvgjmBw1aBgmGn1hjfGyRkmN3krWpFP4",
                "from": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh",
                "hash": "0x3a2b0e21a806c32d79c14d34dd7ee467eb1c4be05fb3dc1b1a58c7b3e3c1a44a"
            },
            "receipt": {
                "ty": 1,
                "tyName": "ExecPack",
                "logs": [
                    {
                        "ty": 1,
                        "tyName": "LogErr",
                        "log": "ErrNoBalance",
                        "rawLog": "0x4572724e6f42616c616e6365"
                    }
                ]
            },
            "height": 60,
            "index": 1,
            "blocktime": 1546070625,
            "amount": "1.0000",
            "fromaddr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh",
            "actionname": "transfer",
            "assets": [
                {
                    "exec": "user.p.evmtest.paracross",
                    "symbol": "coins.bty",
                    "amount": 100000000
                }
            ]
        }
    ]
}

```
 1. 现在trade 只支持主币和资产的交易， 所以交易需要两个步骤
    1. 将paracross里的币 和 对应平行链的主币交易
    1. 在将平行链的主币 和需要的币交易
 2. 将交易所得的币从trade 合约 提到对应的使用的合约， 进行使用
 3. 剩下的币，或和别人交易得到的币从平行链移回主链
```bash
./chain33-cli para asset_withdraw --title user.p.evmtest. -a 0.7 -n test -t 12qyocayNF7Lv6C9qW4avxs2E7U41fKSfv
```
