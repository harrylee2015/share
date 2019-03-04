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

 虚拟机，2核5G的配置，单节点，每秒出一个块，加压后，tps在450左右，趋于稳定后
 统计分析耗时函数
  
   序号|模块|函数名|耗时统计
   ---|---|---|-----
   1|blockchain|CheckTxDup|47.400752ms
   2|exector|ExecTx|46.648793ms
   3|store|ExecKVMemSet|241.862356ms
   4|store|ExecKVSetCommit|134.800212ms
   5|blockchain|ExecBlock|490.091795ms
   
  当数据库是mavl方式时,随着区块的增长，执行时间cost 也会缓慢增长
  
  
  
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

   * 后期优化的重点可以重点围绕CheckTxDup，ExecTx，CheckTxDup 这三个函数进行优化 
