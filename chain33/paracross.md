# 跨链币使用

目前chain33 支持把资产 从主链跨链到平行链使用。 具体步骤如下

 1. 在主链上先把资产转移到paracross 合约。 目前资产有 coins 和 token 的币
 1. 调用 paracross 合约的 asset_transfer 交易， 将币转移到 指定的平行链
    1. 对应的币就会出现在 对应的平行链的 paracross 合约
```bash
./chain33-cli para asset_transfer --title user.p.para. -a 1.4 -n test -t 12qyocayNF7Lv6C9qW4avxs2E7U41fKSfv
```

 1. 调用 平行链的 paracross 合约的 send_exec 将 币 转移到 trade 合约
```bash
./chain33-cli para transfer_exec -a 1.4 -n test -s token.TEST/coins.bty --title user.p.para.guodun. -t trade
```

 1. 现在trade 只支持主币和资产的交易， 所以交易需要两个步骤
    1. 将paracross里的币 和 对应平行链的主币交易
    1. 在将平行链的主币 和需要的币交易
 1. 将交易所得的币从trade 合约 提到对应的使用的合约， 进行使用
 1. 剩下的币，或和别人交易得到的币从平行链移回主链
```bash
./chain33-cli para asset_withdraw --title user.p.para. -a 0.7 -n test -t 12qyocayNF7Lv6C9qW4avxs2E7U41fKSfv
```
