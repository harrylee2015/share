# golang的内存模型

## 前言

go语言内存模型保证同一个变量在不同goroutine被读写到正确的值。一种情况是，变量在一个goroutine中被写入以后，另一个goroutine能够观测到最新写入的结果。


程序中的数据同时被多个goroutine访问（包括读和写）时，这些并发的访问必须以一定的顺序进行。
 
为了将访问序列化，一种方法是采用channel，一种是使用早期的同步控制方式，比如sync、sync/atomic包。

## Part 1：Happens Before（发生在...之前）

为了精确表述读取和写入的顺序，我们定义了happens before（在…之前发生），它是一种go程序内存操作的执行偏序。

HappensBefore：如果事件 e1发生在事件e2之前，那么我们声称事件e2发生在事件e1之后。
 
并发执行：如果事件e1没有发生在事件e2之前，且事件e2没有发生在事件e1之前，那么我们声称事件e1和事件e2并发执行。

在单一goroutine中，happens before偏序就是程序表现出的顺序。

当多个goroutine使用变量 V 时，这些goroutine必须采用同步事件去建立happens before条件，以保证读操作能够观察到需要的写操作。


## Part 2：Synchronization（同步）

2.1 初始化（Initialization）
程序初始化时，只会启动一个goroutine，但是这个goroutine可能会创建其它 goroutine，新创建的goroutine并发执行。
 
如果 package p imports package q，那么q的init函数全部执行结束后，p中的init函数才会开始执行。
 
所有init函数全部执行结束之后，main.main函数才开始执行。

2.2 Goroutine的创建

使用“go”修饰一个语句时，会创建一个新的goroutine。go 修饰的语句执行以后，启动的goroutine才会开始执行。

2.3 Goroutine的销毁

goroutine的退出与程序中的任何事件不存在happens-before关系。

**如果一个goroutine的结果需要被另一个goroutine观察到，必须采用一些同步机制（锁、channel）建立相对顺序。**
