# chain33代码检测分析

## 代码中函数圈复杂度检测

* [圈复杂度检查工具](https://github.com/fzipp/gocyclo)

* 检查命令
```bash
harry@harry-VirtualBox:~/gopath/src/github.com/33cn/chain33$ rm -rf vendor/

harry@harry-VirtualBox:~/gopath/src/github.com/33cn/chain33$ gocyclo -top 50  .
```

* 检查结果  top 50

```go
104 jsonpb (*Unmarshaler).unmarshalValue types/jsonpb/jsonpb.go:740:1
81 client_test (*GrpcCtx).Run client/rpc_ctx_test.go:90:1
56 jsonpb (*Marshaler).marshalObject types/jsonpb/jsonpb.go:177:1
52 client_test (*mockWallet).SetQueueClient client/mock_wallet_test.go:15:1
44 jsonpb (*Marshaler).marshalValue types/jsonpb/jsonpb.go:491:1
43 client_test (*mockBlockChain).SetQueueClient client/mock_blockchain_test.go:17:1
35 commands minerStat system/dapp/commands/stat.go:454:1
34 blockchain (*BlockChain).ProcRecvMsg blockchain/proc.go:18:1
33 jsonpb checkRequiredFields types/jsonpb/jsonpb.go:1196:1
32 p2p (*Cli).GetBlocks p2p/p2pcli.go:391:1
27 main isValidNumber cmd/jsfmt/main.go:102:1
26 blockchain execBlock blockchain/exec.go:17:1
23 p2p (*Node).getAddrFromOnline p2p/monitor.go:91:1
23 commands OneStepSend system/dapp/commands/send.go:17:1
23 wallet (*Wallet).ProcSignRawTx wallet/wallet_proc.go:37:1
22 types (*Transactions).Check types/tx.go:113:1
21 util ExecBlock util/util.go:223:1
21 p2p (*P2pserver).ServerStreamRead p2p/p2pserver.go:441:1
21 merkle Computation common/merkle/merkle.go:53:1
21 commands certGenerate system/dapp/commands/cert.go:77:1
21 mavl TestPruningFirstLevelNode system/store/mavl/db/tree_test.go:1258:1
20 types initForkConfig types/fork.go:271:1
20 p2p (*Node).nodeReBalance p2p/monitor.go:238:1
19 p2p (*Node).monitorDialPeers p2p/monitor.go:407:1
19 wallet (*Wallet).ProcMergeBalance wallet/wallet_proc.go:633:1
19 types _Request_OneofMarshaler types/pbft.pb.go:468:1
19 wallet (*Wallet).ProcCreateNewAccount wallet/wallet_proc.go:220:1
19 log15 escapeString common/log/log15/format.go:243:1
19 p2p (*Peer).readStream p2p/peer.go:295:1
18 types _Request_OneofUnmarshaler types/pbft.pb.go:519:1
18 blockchain (*BlockChain).connectBlock blockchain/process.go:277:1
18 wallet (*Wallet).ProcWalletSetPasswd wallet/wallet_proc.go:743:1
18 commands totalCoins system/dapp/commands/stat.go:64:1
18 wallet (*Wallet).ProcImportPrivKey wallet/wallet_proc.go:379:1
18 types (*BaseCasePack).CheckResult cmd/autotest/types/baseCase.go:214:1
18 p2p (*Peer).sendStream p2p/peer.go:179:1
17 wallet (*Wallet).ProcSendToAddress wallet/wallet_proc.go:485:1
17 commands execBalance system/dapp/commands/stat.go:667:1
17 mavl TestBasic system/store/mavl/db/tree_test.go:81:1
17 executor (*Executor).procExecTxList executor/executor.go:196:1
16 main scanWrite cmd/write/write.go:103:1
16 commands ticketInfoList system/dapp/commands/stat.go:357:1
16 executor (*Executor).procExecAddBlock executor/executor.go:264:1
16 types Init types/config.go:238:1
16 testflow (*TestOperator).HandleDependency cmd/autotest/testflow/flow.go:74:1
16 mavl (*Node).Hash system/store/mavl/db/node.go:159:1
16 strategy (*importPackageStrategy).initData cmd/tools/strategy/importpackage.go:88:1
16 mavl (*Node).traverseInRange system/store/mavl/db/node.go:516:1
16 p2p (*NetAddress).ReachabilityTo p2p/netaddress.go:204:1
16 testflow (*TestOperator).RunCheckFlow cmd/autotest/testflow/flow.go:240:1
```

## 代码中错误流程检测

* [安全检查工具](https://github.com/securego/gosec)
  在线不好安装的话，可以直接离线下载官方发布releases，选择自己需要的安装，https://github.com/securego/gosec/releases

* 执行检查命令
 具体可参考gosec使用说明，这里是
 
```bash
harry@harry-VirtualBox:~/gopath/src/github.com/33cn/chain33$ gosec -nosec=true ./...
```

* 检查结果

```go
...

[/home/harry/gopath/src/github.com/33cn/chain33/wallet/common/store.go:308] - G104: Errors unhandled. (Confidence: HIGH, Severity: LOW)
  > store.GetDB().DeleteSync(CalcLabelKey(label))


[/home/harry/gopath/src/github.com/33cn/chain33/wallet/common/store.go:319] - G104: Errors unhandled. (Confidence: HIGH, Severity: LOW)
  > store.GetDB().SetSync(version.WalletVerKey, data)


Summary:
   Files: 341
   Lines: 89372
   Nosec: 0
  Issues: 735

```
总共扫出了735个问题点，具体需要投入人力对扫出的问题点逐个排查。

这里的风险主要在于系统中存在一些错误没有妥善处理，一旦发生时，可能会导致整个系统奔溃。


## 代码中单元测试检测

* 检测命令

```bash
harry@harry-VirtualBox:~/gopath/src/github.com/33cn/chain33$ go list ./... |grep -v "vendor" |grep -v "mocks" |xargs go test
```
* 检查结果

  略
  
  存在的主要问题是代码覆盖率不足，有的包还没有单元测试
