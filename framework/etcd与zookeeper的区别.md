
# etcd与zookeeper的区别


## 区别

1、两个应用实现的目的不同。etcd的目的是一个高可用的 Key/Value 存储系统，主要用于分享配置和服务发现；zookeeper的目的是高有效和可靠的协同工作系统。

2、接口调用方式不同。etcd是基于HTTP+JSON的API，直接使用curl就可以轻松使用，方便集群中每一个主机访问；zookeeper基于TCP，需要专门的客户端支持。

3、功能就比较相似了。etcd和zookeeper都是提供了key，value存储服务，集群队列同步服务，观察一个key的数值变化。

4、部署方式也是差不多：采用集群的方式，可以达到上千节点。只是etcd是go写的，直接编译好二进制文件部署安装即可；zookeeper是java写的，需要依赖于jdk，需要先部署jdk。

5、实现语言： go 拥有几乎不输于C的效率，特别是go语言本身就是面向多线程，进程通信的语言。在小规模集群中性能非常突出；java，实现代码量要多于go，在小规模集群中性能一般，但是在大规模情况下，使用对多线程的优化后，也和go相差不大。


## 总结

1、在原生接口和提供服务方式方面，etcd更适合作为集群配置服务器，用来存储集群中的大量数据，基于HTTP+JSON的API可以让集群中的任意一个节点在使用key/value服务时获取方便；

2、zookeeper则更加的适合于提供分布式协调服务，在实现分布式锁模型方面较etcd要简单的多。

实际使用中应该根据自身使用情况来选择相应的服务。
