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
harry@harry-VirtualBox:~/parachain$ ./chain33-cli send  para asset_transfer --title user.p.guodun. -a 10  -n test -t "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh" -k "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
0x1d9d893914b3f6e631abaa33b7d57cc72ff05457d3b482ec2d1ae0907955b4af

```
* 结果
```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli asset balance --addr "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh" --asset_exec paracross coins  --asset_symbol coins.bty  -e  "paracross"   --rpc_laddr http://localhost:8901
{
    "balance": "10.0000",
    "frozen": "0.0000",
    "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
}

```

 3. 调用平行链的 paracross 合约的 send_exec 将 币 转移到 trade 合约
 
  * 转币
 
```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli para transfer_exec -a 5 -e user.p.guodun.trade  -s coins.bty  --rpc_laddr http://localhost:8901
```
```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli wallet sign -d xxxx  -a 16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh  -e 300s -f 0.001 -k 16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh  --paraName user.p.guodun. --rpc_laddr http://localhost:8901
```
```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli wallet send -d xxxxx --paraName user.p.guodun. --rpc_laddr http://localhost:8901
```


* 结果查询

```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli asset balance -a 16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh --asset_exec paracross coins  --asset_symbol coins.bty -e user.p.guodun.trade   --rpc_laddr http://localhost:8901
{
    "balance": "5.0000",
    "frozen": "0.0000",
    "addr": "16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh"
}
```

 1. 现在trade 只支持主币和资产的交易， 所以交易需要两个步骤
    1. 将paracross里的币 和 对应平行链的主币交易
    
    ```bash
    harry@harry-VirtualBox:~/parachain$ curl -d '{"jsonrpc":"2.0", "id": 1, "method":"trade.CreateRawTradeSellTx","params":[{"tokenSymbol":"coins.bty", "amountPerBoardlot":1, "minBoardlot":1,"pricePerBoardlot":1,"totalBoardlot":15000000,"fee":100000,"assetExec":"paracross"}]}'         -H 'content-type:text/plain;'         http://localhost:8901
    ```
    ```bash
harry@harry-VirtualBox:~/parachain$ ./chain33-cli wallet sign -d xxxx   -a 16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh  -e 300s -f 0.001 -k 16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh  --paraName user.p.guodun. --rpc_laddr http://localhost:8901
  ```
    
  查看卖单
    
  ```bash
 harry@harry-VirtualBox:~/parachain$ curl  -X POST  http://localhost:8901 -H "Content-Type:application/json" -d '{"jsonrpc":"2.0", "id": 1, "method":"Chain33.Query","params":[{"execer":"trade", "funcName":"GetTokenSellOrderByStatus", "payload" : {"tokenSymbol": "coins.bty", "status": 1, "count":10, "direction": 0,"fromKey":""}}]}' 
   ```
   
   ```json
{"id":1,"result":{"orders":[{"tokenSymbol":"coins.bty","owner":"16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh","amountPerBoardlot":"1","minBoardlot":"1","pricePerBoardlot":"1","totalBoardlot":"15000000","tradedBoardlot":"0","buyID":"","status":1,"sellID":"mavl-trade-sell-48ca0988435ae210e35e9a9f33b40a0a07968bd87af5170c2352790563670bed","txHash":"0x48ca0988435ae210e35e9a9f33b40a0a07968bd87af5170c2352790563670bed","height":"50","key":"mavl-trade-sell-48ca0988435ae210e35e9a9f33b40a0a07968bd87af5170c2352790563670bed","blockTime":"1547516548","isSellOrder":true,"assetExec":"paracross"}]} ,"error":null}
    ```
    1. 在将平行链的主币 和需要的币交易
    
    ```bash
    ```
 2. 将交易所得的币从trade 合约 提到对应的使用的合约， 进行使用
 3. 剩下的币，或和别人交易得到的币从平行链移回主链
```bash
./chain33-cli para asset_withdraw --title user.p.evmtest. -a 0.7 -n test -t 12qyocayNF7Lv6C9qW4avxs2E7U41fKSfv
```
