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
0a17757365722e702e67756f64756e2e7061726163726f7373124e1004424a0a09636f696e732e6274791080cab5ee012213757365722e702e67756f64756e2e74726164652a223148557a4e7952774b6d38576d46576b456964626f6f6654374478707a62694b583120a08d0630e7f98e93fc97a186033a223145586d42724347775a61545a754c72777073487a384247365062565856317a6259
harry@harry-VirtualBox:~/parachain$ ./chain33-cli wallet sign -d 0a17757365722e702e67756f64756e2e7061726163726f7373124e1004424a0a09636f696e732e6274791080cab5ee012213757365722e702e67756f64756e2e74726164652a223148557a4e7952774b6d38576d46576b456964626f6f6654374478707a62694b583120a08d0630e7f98e93fc97a186033a223145586d42724347775a61545a754c72777073487a384247365062565856317a6259  -a 16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh  -e 300s -f 0.001 -k 16ReZHzMCGtPt8B7XbnZQ2jeXsPG9wEufh  --paraName user.p.guodun. --rpc_laddr http://localhost:8901
0a17757365722e702e67756f64756e2e7061726163726f7373124e1004424a0a09636f696e732e6274791080cab5ee012213757365722e702e67756f64756e2e74726164652a223148557a4e7952774b6d38576d46576b456964626f6f6654374478707a62694b58311a6e08011221021832ee50ab60016d6f6bf03315b9354434ba487929c57c79d67de2309aba3bfe1a473045022100f510a02de777e50161c0d601c1a8261fd74cc6ea8e1abc054427916cca8d0c8e0220483c638b12b711f10200b2d563024826ec15b2e30d66c0078359815a5411d38920a08d062899daf1e10530e7f98e93fc97a186033a223145586d42724347775a61545a754c72777073487a384247365062565856317a6259
harry@harry-VirtualBox:~/parachain$ ./chain33-cli wallet send -d 0a17757365722e702e67756f64756e2e7061726163726f7373124e1004424a0a09636f696e732e6274791080cab5ee012213757365722e702e67756f64756e2e74726164652a223148557a4e7952774b6d38576d46576b456964626f6f6654374478707a62694b58311a6e08011221021832ee50ab60016d6f6bf03315b9354434ba487929c57c79d67de2309aba3bfe1a473045022100f510a02de777e50161c0d601c1a8261fd74cc6ea8e1abc054427916cca8d0c8e0220483c638b12b711f10200b2d563024826ec15b2e30d66c0078359815a5411d38920a08d062899daf1e10530e7f98e93fc97a186033a223145586d42724347775a61545a754c72777073487a384247365062565856317a6259  --paraName user.p.guodun. --rpc_laddr http://localhost:8901
0x7fed301a98d3e202d1e5fe15eb57d2c8c64c597f5bf469a1698b3b748c1dce4c
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

    ```
    1. 在将平行链的主币 和需要的币交易
 2. 将交易所得的币从trade 合约 提到对应的使用的合约， 进行使用
 3. 剩下的币，或和别人交易得到的币从平行链移回主链
```bash
./chain33-cli para asset_withdraw --title user.p.evmtest. -a 0.7 -n test -t 12qyocayNF7Lv6C9qW4avxs2E7U41fKSfv
```
