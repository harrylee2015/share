# RabbitMQ

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
