# chain33中exector中benchmark性能测试分析

**将其他benchmark函数注释掉，只保留BenchmarkExecBlock**

* 执行
```bash
go test ./executor/... -v -bench=.
```

* 执行结果：
```bash
BenchmarkExecBlock-2   	       1	3194754533 ns/op
PASS
ok  	github.com/33cn/chain33/executor	6.676s

```
