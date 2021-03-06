# RabbitMQ
<!-- TOC -->

- [RabbitMQ](#rabbitmq)
  - [RabbitMQ简介](#rabbitmq简介)
  - [RabbitMQ的特点](#rabbitmq的特点)
  - [RabbitMQ的工作模式](#rabbitmq的工作模式)
  - [RabbitMQ的四种交换机类型](#rabbitmq的四种交换机类型)

<!-- /TOC -->


## RabbitMQ简介
 
  * RabbitMQ 是一个消息中间件 , 一个由 Erlang 语言开发的 AMQP 的开源实现。

  * AMQP : Advanced Message Queue Protocol，高级消息队列协议。它是应用层协议的一个开放标准，为面向消息的中间件设计，基于此协议的客户端与消息中间件可传递消息，并不受产品、开发语言等条件的限制。


## RabbitMQ的特点

RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。具体特点如下：

  * 可靠性（Reliability） : RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认。
  
  * 灵活的路由（Flexible Routing） : 在消息进入队列之前，通过 Exchange 来路由消息的。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也通过插件机制实现自己的 Exchange 。

  * 消息集群（Clustering） : 多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。

  * 高可用（Highly Available Queues） : 队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。

  * 多种协议（Multi-protocol） : RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。

  * 多语言客户端（Many Clients） : RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。

  * 管理界面（Management UI）: RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。

  * 跟踪机制（Tracing）: 如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。
  
  * 插件机制（Plugin System）: RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件


## RabbitMQ的工作模式

   1. simple简单模式

      消息产生着§将消息放入队列
      
      消息的消费者(consumer) 监听(while) 消息队列,如果队列中有消息,就消费掉,消息被拿走后,自动从队列中删除(隐患 消息可能没有被消费者正确处理,已经从队列中消失了,造成消息的丢失)
      
      **应用场景:聊天(中间有一个过度的服务器;p端,c端)**
      
      
   2. work工作模式(资源的竞争)

      消息产生者将消息放入队列消费者可以有多个,消费者1,消费者2,同时监听同一个队列,消息被消费?C1 C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患,高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize,与同步锁的性能不一样) 保证一条消息只能被一个消费者使用)
      
      **应用场景:红包;大项目中的资源调度(任务分配系统不需知道哪一个任务执行系统在空闲,直接将任务扔到消息队列中,空闲的系统自动争抢)**
      
   3. publish/subscribe发布订阅(共享资源)

      消息产生者将消息放入交换机,交换机发布订阅把消息发送到所有消息队列中,对应消息队列的消费者拿到消息进行消费
         
      **相关场景:邮件群发,群聊天,广播(广告)**
     
   4. routing路由模式


      消息生产者将消息发送给交换机按照路由判断,路由是字符串(info) 当前产生的消息携带路由字符(对象的方法),交换机根据路由的key,只能匹配上路由key对应的消息队列,对应的消费者才能消费消息;

      根据业务功能定义路由字符串


   5. topic 主题模式(路由模式的一种)
   
   
   
   6. RPC模式
   
       **远程回调函数**


      [golang中测试用例](https://github.com/harrylee2015/harry_tools/tree/master/docker/rabbitmq)

## RabbitMQ的四种交换机类型

   1. direct
   
      指定路由
      
   2. fanout
   
      广播绑到到同一个交换机上面的所有Queue消息
   
   3. topic
      
      订阅模式，一对多
      
   4. headers
   
      不处理路由键。而是根据发送的消息内容中的headers属性进行匹配。在绑定Queue与Exchange时指定一组键值对；当消息发送到RabbitMQ时会取到该消息的headers与Exchange绑定时指定的键值对进行匹配；如果完全匹配则消息会路由到该队列，否则不会路由到该队列。headers属性是一个键值对，可以是Hashtable，键值对的值可以是任何类型。而fanout，direct，topic 的路由键都需要要字符串形式的。匹配规则x-match有下列两种类型：

      x-match = all ：表示所有的键值对都匹配才能接受到消息

      x-match = any ：表示只要有键值对匹配就能接受到消息
   
     
