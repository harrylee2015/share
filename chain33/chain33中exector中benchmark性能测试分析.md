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
