# chain33 queue 模块

<!-- TOC -->

- [queue模块](#mempool模块)
    - [1. 模块介绍](#1-模块介绍)
    - [2. 逻辑架构及上下文](#2-逻辑架构及上下文)
    - [3. queue和client数据结构](#3-queue和client数据结构)
        - [3.1 queue的数据结构和接口](#31-queue的数据结构和接口)
	- [3.2 chanSub订阅通道和Message](#32-chanSub订阅通道和Message)
	- [3.3 Client数据结构和接口](#33-Client数据结构和接口)
	- [3.4 Module接口](#34-Module接口)
    - [4. chain33中queue的使用](#4-chain33中queue的使用)

<!-- /TOC -->

##  1. 模块介绍
> queue 顾名思义，这个模块实现了消息队列的功能。
> 主要目的是为了实现chain33系统中各个模块的解耦，每个模块相对来说独立，不是通过接口调用，而是通过消息进行通信，
> 为后面各个模块的微服务化做准备。



##  2. 逻辑架构及上下文
> 下图就是各模块与queue模块的关联关系

![queue 模块交互架构图](resource/queue.png)

> 注意:

>  1.  在我们chain33中，系统只实例化一个queue，每个模块都实例化了一个queue.client

>  2.  一个queue可以对应多个订阅主题，也就是一个queue可以实例化多个queue.client

>  3.  一个queue.client只能对应一个订阅主题。

>  4.  exector和rpc模块各自都额外实例化一个QueueProtocol


## 3. queue和client数据结构

> queue 中主要定义了三种数据结构，queue, client, Message
>
> 下面分别介绍他们的数据结构及相应的接口

### 3.1 queue的数据结构和接口

> queue这里的实现主要是利用go语言中channel的属性，利用缓存通道先进先出的特性来实现。
> 定义了chanSub定义通道，一个订阅主题对应一个chanSub.

```go
// queue数据结构
type queue struct {
	chanSubs map[string]*chanSub   // queue中订阅的通道 key=订阅名，value怎是订阅通道
	mu       sync.Mutex            // 同步锁
	done     chan struct{}         // 结束通知
	interupt chan struct{}         // 当程序被打断时，如linux中用ctrl+c结束运行程序，会触发interupt去关闭queue
	isClose  int32                 // queue状态，1为关闭状态
	name     string                // 队列名称
}
```

> queue接口比较简单，最主要的是client()这个方法，后面所有操作都是以client方式去执行
> queue与client是一对多的关系。

```go
// queue接口
type Queue interface {
	Close()                       // 关闭queue
	Start()                       // 启动
	Client() Client               // 获取queue的客户端，在chain33中，所有msg的 send和receive都通过client去调用
	Name() string                 // 获取queue名称
}
```

### 3.2 chanSub订阅通道和Message

> chanSub中我们根据channel缓存大小,设置了一个同步，和异步通道
> 同步通道中的数据处理优先级最高

```go
// chanSub数据结构
type chanSub struct {
	high    chan Message          // 同步处理通道，默认缓存大小64
	low     chan Message          // 异步处理通道，默认缓存大小40960
	isClose int32                 // 订阅通道状态 1为关闭
}
```

> 这里我们定义了Msg,也巧妙地利用了chan属性，
> 将相应数据通过chReply响应通道返回。

```go
// Message数据结构
type Message struct {
	Topic   string                // 主题，也就是给谁发送的
	Ty      int64                 // 消息类型
	Id      int64                 // 自增长的Id,在同一个queue中，每个msg有唯一的Id与之对应
	Data    interface{}           // 消息数据
	chReply chan Message          // msg响应通道，用于返回接收方响应的数据，这里比较巧妙地利用了chan特有的属性
}
```

### 3.3 Client数据结构和接口

> queue中的所有msg的send,receive 都是通过client去发送

```go
// Client数据结构
type client struct {
	q          *queue             // queue实例
	recv       chan Message       // 接收Msg通道，默认缓冲时是5
	done       chan struct{}      // 用来通知client关闭
	wg         *sync.WaitGroup    // 同步等待
	topic      string             // 订阅主题
	isClosed   int32              // client状态，1为关闭
	isCloseing int32              // 1表示client状态关闭
}
```

> client实现了如下方法，构造Msg,send,wait,Recv等

```go
// Client接口
type Client interface {
	Send(msg Message, waitReply bool) (err error)                               //发送消息，默认主网超时时间是10分钟
	SendTimeout(msg Message, waitReply bool, timeout time.Duration) (err error) //发送消息，自定义超时时间
	Wait(msg Message) (Message, error)                                          //等待消息处理完成，默认主网超时时间是10分钟
	WaitTimeout(msg Message, timeout time.Duration) (Message, error)            //等待消息处理完成，自定义超时时间
	Recv() chan Message                                                         //获取client接收消息通道
	Sub(topic string)                                                           //订阅消息
	Close()                                                                     //关闭client
	CloseQueue() (*types.Reply, error)                                          //关闭queue
	NewMessage(topic string, ty int64, data interface{}) (msg Message)          //new 一个msg
}
```


### 3.4 Module接口

> chain33中所有模块都实现了Module接口
> 其中SetQueueClient()方法主要功能是设置订阅主题，根据业务处理自己订阅的Msg并返回.

```go
// Module be used for module interface
type Module interface {
	SetQueueClient(client Client) //各个模块都会实现这个接口，在这里面会对相应的msg做具体的处理
	Close()
}

```


## 4. chain33中queue的使用

>  下面将结合chain33中的代码，以伪代码的形式给大家呈现整个流程


### 4.1 初始化queue，赋值并启动

```go
func main(){
    ......
    log.Info("loading queue")
	q := queue.New("channel")        //实例化一个名为channel的queue

	log.Info("loading blockchain module")
	chain := blockchain.New(cfg.BlockChain)
	chain.SetQueueClient(q.Client()) //将q.client()传入,SetQueueClient()是各个模块实际的处理逻辑
    ......
    //jsonrpc, grpc, channel 三种模式
	rpcapi := rpc.New(cfg.Rpc)
	rpcapi.SetQueueClient(q.Client()) //给rpcapi赋值client
	......
    q.Start()                        //启动queue
}
```

### 4.2 实现Moudle接口，进行具体业务处理

> 这里以blockchain模块为例，展示如何处理收到的msg

> blockchain实现SetQueueClient方法

```go
func (chain *BlockChain) SetQueueClient(client queue.Client) {
	chain.client = client
	chain.client.Sub("blockchain")  //设置订阅主题，本client会只接收topic为blockchain的消息
    ......
	go chain.ProcRecvMsg()          //对接收到的消息做具体的处理
}
```

> blockchain对接收msg进行处理

```go
//blockchain模块的消息接收处理
func (chain *BlockChain) ProcRecvMsg() {
	defer chain.recvwg.Done()
	reqnum := make(chan struct{}, 1000)  //这里做了流量控制，最多同时处理1000条msg
	for msg := range chain.client.Recv() { //从client中取msg
		chainlog.Debug("blockchain recv", "msg", types.GetEventName(int(msg.Ty)), "id", msg.Id, "cap", len(reqnum))
		msgtype := msg.Ty
		reqnum <- struct{}{}
		atomic.AddInt32(&chain.runcount, 1)
		switch msgtype {
		case types.EventLocalGet:
			go chain.processMsg(msg, reqnum, chain.localGet) // blockchain中msg的通用处理方法

        ......
	}
}
```

> processMsg方法处理逻辑，调用传入的func

```go
// processMsg
func (chain *BlockChain) processMsg(msg queue.Message, reqnum chan struct{}, cb funcProcess) {
	beg := types.Now()
	defer func() {
		<-reqnum 
		atomic.AddInt32(&chain.runcount, -1) //处理完一条msg,runcount计数器就减一
		chainlog.Debug("process", "cost", types.Since(beg), "msg", types.GetEventName(int(msg.Ty)))
	}()
	cb(msg) //调用传过来的函数
}
```

> 根据消息类型触发相应的func,这里是localGet方法

```go
// localGet
func (chain *BlockChain) localGet(msg queue.Message) {
	keys := (msg.Data).(*types.LocalDBGet)
	values := chain.blockStore.Get(keys)
	msg.Reply(chain.client.NewMessage("rpc", types.EventLocalReplyValue, values))
}
```
> 返回处理后的结果

```go
// Message中的Reply方法
func (msg Message) Reply(replyMsg Message) {
	if msg.chReply == nil {
		qlog.Debug("reply a empty chreply", "msg", msg)
		return
	}
	msg.chReply <- replyMsg  //就是将replyMsg塞给接收的msg的chReply通道
	qlog.Debug("reply msg ok", "msg", msg)
}
```

### 4.3 QueueProtocol对client进行二次包装，满足外部调用

> 从上面我们可以知道msg的send和receive都需要通过queue的client
> 这在系统内部模块如block,mempool,consensus等，可以直接调用client.Send()方法就可以了
> 但是如何我是在系统外部，该如何调用呢？不急，我们先了解一下QueueProtocol

#### 4.3.1 QueueProtocol数据结构

> QueueProtocol数据结构，这里主要是对queue.client进行包装，无需设置订阅主题，
> 就可以和其他模块进行通信。

```go
// QueueProtocol数据结构
type QueueProtocolOption struct {
	// 发送请求超时时间
	SendTimeout time.Duration
	// 接收应答超时时间
	WaitTimeout time.Duration
}

// 消息通道协议实现
type QueueProtocol struct {
	// 消息队列
	client queue.Client
	option QueueProtocolOption
}
```

#### 4.3.2 RPC和Executor中实例化QueueProtocol

> 聪明的你一定猜到怎么调用了。你想的没错，就是通过实例化QueueProtocol去调用
> 这里最终会实例化一个QueueProtocol对象,client.New(q.Client(), nil)，jsonRPC和gRPC
> 共享这一个QueueProtocol

> RPC模块实现SetQueueClient方法

```go
// rpc 实现 Module接口
func (r *RPC) SetQueueClient(c queue.Client) {
	gapi := NewGRpcServer(c)
	japi := NewJSONRPCServer(c)
	r.gapi = gapi
	r.japi = japi
	r.c = c
	//注册系统rpc
	pluginmgr.AddRPC(r)
	go gapi.Listen()
	go japi.Listen()
}
```
> 执行器模块也实例化一个QueueProtocol

```go
func (exec *Executor) SetQueueClient(qcli queue.Client) {
	exec.client = qcli
	exec.client.Sub("execs")
	var err error
	exec.qclient, err = client.New(qcli, nil)  //执行器模块也实例化一个QueueProtocol
	if err != nil {
		panic(err)
	}
	//recv 消息的处理
	go func() {
		for msg := range exec.client.Recv() {
		.....
	}()
}
```

#### 4.3.3 QueueProtocolAPI实现的QueueProtocolAPI接口

> QueueProtocolAPI中主要定义并实现了一些API方法，供其他模块调用，主要就是将
> API方法传入的参数转化为相应的msg发送，接收并处理返回的msg,将msg再转化为
> API方法所需的数据结构。

```go
// 消息通道交互API接口定义
type QueueProtocolAPI interface {
    
    ......
	// +++++++++++++++ consensus interfaces begin
	// types.EventGetTicketCount
	GetTicketCount() (*types.Int64, error)
	// --------------- consensus interfaces end

	// +++++++++++++++ wallet interfaces begin
	// types.EventLocalGet
	LocalGet(param *types.LocalDBGet) (*types.LocalReplyValue, error)
	// types.EventLocalList
    ......
}
```

```go

```
> 这里以LocalGet()举例，展示如何将参数转化为msg处理

> 调用QueueProtocol中的LocalGet()方法，去触发往blockchain中发送types.EventLocalGet类型的 msg
 
> LocalGet方法触发query方法

```go
func (q *QueueProtocol) LocalGet(param *types.LocalDBGet) (*types.LocalReplyValue, error) {
    ......
	msg, err := q.query(blockchainKey, types.EventLocalGet, param)  //QueueProtocol模块通用的query方法
	if err != nil {
		log.Error("LocalGet", "Error", err.Error())
		return nil, err
	}
	if reply, ok := msg.GetData().(*types.LocalReplyValue); ok { 
		return reply, nil
	}
	......
	return nil, types.ErrTypeAsset
}
```

> query查询方法，就是将相关param包装成Msg,发送到对应的模块。

```go
func (q *QueueProtocol) query(topic string, ty int64, data interface{}) (queue.Message, error) {
	client := q.client
	msg := client.NewMessage(topic, ty, data) //将相关参数转化为msg
	var trace *tlog.TraceInfo
	if tlog.TraceOn() {
		if item, ok := data.(*types.Transaction); ok {
			trace = tlog.CreateTraceMsg(msg.Id, ty, hex.EncodeToString(item.Hash()), tlog.MsgCreate, tlog.RpcKey, "query")
			if trace != nil {
				trace.Begin()
			}
		}
	}
	err := client.SendTimeout(msg, true, q.option.SendTimeout) //同步发送
	if err != nil {
		if trace != nil {
			trace.Error(err)
		}
		return queue.Message{}, err
	}
	msg, errR := client.WaitTimeout(msg, q.option.WaitTimeout) //等待处msg处理完后响应
	if trace != nil {
		trace.UpdateState(tlog.MsgProcessed)
		if errR != nil {
			trace.Error(errR)
		} else {
			trace.End()
		}
	}
	return msg, errR
}
```
> client中的waitTimeout方法，等待msg的响应结果

```go
// client中 waitTimeout方法
func (client *client) WaitTimeout(msg Message, timeout time.Duration) (Message, error) {
	if msg.chReply == nil {
		return Message{}, errors.New("empty wait channel")
	}
	t := time.NewTimer(timeout)
	defer t.Stop()
	select {
	case msg = <-msg.chReply:  //对原有的msg进行重新赋值，值是从chReply响应通道中取的
		return msg, msg.Err()
	case <-client.done:
		return Message{}, errors.New("client is closed")
	case <-t.C:
		return Message{}, types.ErrTimeout   //如果超时则返回超时错误
	}
}
```
