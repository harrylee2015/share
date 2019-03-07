# chain33中exector中benchmark性能测试分析

**整体所有Benchmark函数执行**

```bash
go test ./executor/... -v -bench=.
```
* 执行结果

```bash
  300000	      3762 ns/op
PASS
ok  	github.com/33cn/chain33/executor	11.134s

```

**分开执行Benchmark函数，记录结果**

* 执行BenchmarkGenRandBlock
```bash
go test -run=exector_test.go  -bench=BenchmarkGenRandBlock

```
* 执行结果：
```bash
goos: linux
goarch: amd64
pkg: github.com/33cn/chain33/executor
BenchmarkGenRandBlock-2   	       1	2251794883 ns/op
PASS
ok  	github.com/33cn/chain33/executor	2.305s

```
* 执行BenchmarkExecBlock
```bash
go test -run=executor_real_test.go  -bench=BenchmarkExecBlock
```

* 执行结果：
```bash
goos: linux
goarch: amd64
pkg: github.com/33cn/chain33/executor
BenchmarkExecBlock-2   	       1	3519987151 ns/op
PASS
ok  	github.com/33cn/chain33/executor	3.609s

```
* 执行BenchmarkLocalDBGet
```bash
go test -run=localdb_test.go  -bench=BenchmarkLocalDBGet
```

* 执行结果：
```bash
  300000	      4307 ns/op
PASS
ok  	github.com/33cn/chain33/executor	3.825s
```


加了打印日志分析各个函数耗时

函数名称|耗时
-------|------
BenchmarkExecBlock|3.00784231s
BenchmarkGenRandBlock|2.121537785s
BenchmarkLocalDBGet|217.909883ms
CreateCoinsBlock|2.169689496s
ExecBlock|618.394741ms
CheckTxDup|22.92537ms
ExecTx|87.043018ms
ExecKVMemSet|320.684168ms
ExecKVSetCommit|548.048394ms

ExecBlock中 ExecTx 和 ExecKVMemSet，ExecKVSetCommit 耗时比较明显

## 对ExecBlock函数，进行单独测试，分析内部相关模块执行耗时

 ### store配置为 mavl 模式

 虚拟机，2核5G的配置，单节点，每秒出一个块，加压后，tps在450左右，趋于稳定后
 统计分析耗时函数
  
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckTxDup|47.400752ms
   2|exector|ExecTx|46.648793ms
   3|store|ExecKVMemSet|241.862356ms
   4|store|ExecKVSetCommit|134.800212ms
   5|blockchain|ExecBlock|490.091795ms
   
   随着区块的增长，执行时间cost 也会缓慢增长
  
  
  
  ### store配置为 kvmvcc 模式
   
   虚拟机，2核5G的配置，单节点，每秒出一个块，加压后，tps在700左右趋于稳定后，
   统计分析耗时函数
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckTxDup|35.366131ms
   2|exector|ExecTx|56.456606ms
   3|store|ExecKVMemSet|3.615731ms
   4|store|ExecKVSetCommit|22.147875ms
   5|blockchain|ExecBlock|132.450229ms
   
   * 通过过对比我们可以发现采用 kvmvcc存储后，ExecKVMemSet和ExecKVSetCommit 两个函数的执行耗时显著下降，同样的配置，系统的吞吐量
   也有所提升

   * 后期优化的重点可以考虑围绕CheckTxDup，ExecTx，ExecKVSetCommit 这三个函数进行优化 
   
   
   
   
   ### store配置为 kvmvcc模式，数据统计
    
   **前提：**
   
   以下均是用norm执行器，每笔交易都是随机去写入100字节KV数据 
   机器配置为10核，32G内存，磁盘为固态硬盘，操作系统ubuntu
    
   
   **以下是raft默认一秒打一个块的测试数据，且TxHeight=false**
   
   * 每个块打包得交易数为 600
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckTxDup|43.871892ms
   2|exector|ExecTx|36.973969ms
   3|merkle|CalcMerkleRoot|2.562651ms
   4|store|ExecKVMemSet|1.397479ms
   5|store|ExecKVSetCommit|7.736843ms
   6|blockchain|ExecBlock|94.214597ms
     
    
   * 每个块打包得交易数为 1500
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckTxDup|136.536985ms
   2|exector|ExecTx|107.269925ms
   3|merkle|CalcMerkleRoot|6.753266ms
   4|store|ExecKVMemSet|3.525295ms
   5|store|ExecKVSetCommit|15.964752ms
   6|blockchain|ExecBlock|274.03935ms
  
  
   * 每个块打包得交易数为 3000 
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckTxDup|277.589837ms
   2|exector|ExecTx|271.341151ms
   3|merkle|CalcMerkleRoot|12.81075ms
   4|store|ExecKVMemSet|6.222214ms
   5|store|ExecKVSetCommit|83.285137ms
   6|blockchain|ExecBlock|658.070058ms
   
   
   * 每个块打包得交易数为 5000 
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckTxDup|611.882472ms
   2|exector|ExecTx|518.891374ms
   3|merkle|CalcMerkleRoot|38.277755ms
   4|store|ExecKVMemSet|11.412032ms
   5|store|ExecKVSetCommit|38.494543ms
   6|blockchain|ExecBlock|1.232125534s

   
   * 每个块打包得交易数为 10000
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckTxDup|1.28111507s
   2|exector|ExecTx|1.193757552s
   3|merkle|CalcMerkleRoot|50.94395ms
   4|store|ExecKVMemSet|26.701296ms
   5|store|ExecKVSetCommit|110.768568ms
   6|blockchain|ExecBlock|2.688707323s
   
   
   外加一秒等待时间，这样一估算，单节点的情况下，此时系统的实际tps只有2000左右
   
   

   **以下是把raft默认的打包等待时间去掉的测试数据**
   
   
   * 每个块打包得交易数为 10左右时
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckSign|1.956µs
   2|blockchain|CheckTxDup|632.067µs
   3|merkle|CalcMerkleRootCache|39.461µs
   4|exector|ExecTx|973.593µs
   5|merkle|CalcMerkleRoot|58.108µs
   6|store|ExecKVMemSet|134.585µs
   7|store|ExecKVSetCommit|3.499766ms
   8|blockchain|ExecBlock|5.918733ms
   
   * 每个块打包得交易数为 20左右时
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckSign|2.235µs
   2|blockchain|CheckTxDup|522.664µs
   3|merkle|CalcMerkleRootCache|422.517µs
   4|exector|ExecTx|1.800693ms
   5|merkle|CalcMerkleRoot|76.752µs
   6|store|ExecKVMemSet|91.627µs
   7|store|ExecKVSetCommit|3.22084ms
   8|blockchain|ExecBlock|7.105836ms
   
   
   * 每个块打包得交易数为 100左右时
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckSign|2.095µs
   2|blockchain|CheckTxDup|2.719762ms
   3|merkle|CalcMerkleRootCache|390.333µs
   4|exector|ExecTx|13.082559ms
   5|merkle|CalcMerkleRoot|782.553µs
   6|store|ExecKVMemSet|583.51µs
   7|store|ExecKVSetCommit|8.227311ms
   8|blockchain|ExecBlock|26.898224ms
   
   
   * 每个块打包得交易数为2000左右时
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckSign|2.305µs
   2|blockchain|CheckTxDup|108.72353ms
   3|merkle|CalcMerkleRootCache|4.237428ms
   4|exector|ExecTx|266.02094ms
   5|merkle|CalcMerkleRoot|10.712475ms
   6|store|ExecKVMemSet|5.801295ms
   7|store|ExecKVSetCommit|19.436243ms
   8|blockchain|ExecBlock|418.780469ms
   
   * 每个块打包得交易数为5000左右时
   
   
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckSign|1.746µs
   2|blockchain|CheckTxDup|450.19369ms
   3|merkle|CalcMerkleRootCache|9.727373ms
   4|exector|ExecTx|726.817774ms
   5|merkle|CalcMerkleRoot|23.826088ms
   6|store|ExecKVMemSet|12.768735ms
   7|store|ExecKVSetCommit|359.795668ms
   8|blockchain|ExecBlock|1.588617707s
   

   这样一估算，单节点的情况下，系统的实际tps在4000到5000左右
