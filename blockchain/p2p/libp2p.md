# libp2p 学习

##  前言 

Libp2p 是一个便于使用者开发去中心化点对点应用的网络框架，它原先是 IPFS 的网络协议延申，如今已自成一家。
所有的分布式点对点网络，都会面临一系列不同于传统网络的挑战。Libp2p 是个通用工具包，有了它，开发者就可以在分布式应用上使用可插拔的网络。

## 实现的功能

**支持各种各样的传输方式：**

- 传输：TCP，UDP，SCTP，UDP，uTP，QUIC，SSH，etc.

- 安全传输：TLS，DTLS，CurveCP，SSH

- 有效使用sockets（连接重用）

- 允许端点之间的交流可以在一个socket上复用（避免过多的握手）

- 允许端点之间通过一个协商过程使用多协议以及各自的版本

- 服务发现（mdns,dht）

- http请求代理，http-proxy

- 实现NAT转换 

- 实现连接中继

- 实现加密通道

- 充分使用基础传输（例如原生的流复用等）


## p2p相关技术博客

 [KAD分布式哈希协议原理](http://www.yeolar.com/note/2010/03/21/kademlia/#id12)
 
 [Chory算法介绍](http://www.yeolar.com/note/2010/04/06/p2p-chord/)
 
 [gossip p2p协议](https://zhuanlan.zhihu.com/p/41228196)
