# Harmony区块链的分片路线



Harmony的目标是打造一个基于分片的区块链，具备完全扩展性、安全性。它研究了市面上很多的区块链解决方案，提出了自己的工程落地方案。
这个是很高的目标，首先具备完全的可扩展性，Harmony的分片不仅包括交易确认、网络通信，也包括区块链状态的分片。其次要保证分片的安全性。Harmony的分片基于DRG（分布式随机生成）过程，这让它具有无法被预测、公平、可验证和可扩展的特性。此外，Harmony采用了PoS机制，而不是PoW机制来选择验证者，它对PBFT共识机制有自己的优化。PoS有一定的门槛，既要保证小的权益质押者能够参与网络和赚取收益，也要防止恶意攻击者在单个分片获得掌控权。Harmony通过采用自适应信息扩散算法（Adaptive Information Dispersal Algorithm）实现分片内和跨分片网络的信息传播。Harmony还采用Kademlia路由实现跨分片交易随着分片数量增加呈对数级扩展。有了分片，还必须保持跨分片交易的一致性，Harmony也支持跨分片交易，支持分片之间的直接通信，通过原子锁定机制确保跨分片交易的一致性。

[harmony团队实现的go版本的纠错码库](https://github.com/harmony-one/go-raptorq)

[harmony团队实现的VDF可验证延迟函数](https://github.com/harmony-one/vdf)




## 参考链接
[蓝狐笔记--Harmony区块链的分片扩展之路](https://mp.weixin.qq.com/s?__biz=MzAwOTk1NjM0NQ==&mid=2247487043&idx=1&sn=fc2d14e7d4adcf82fae7cacfe529a71e&chksm=9b56f0d5ac2179c375b0e6a862ab6458c24042fdc09bc3bcd963c25e0362cd0d35a44ba63aa9&mpshare=1&scene=1&srcid=0703y4Lc6ylXDUTmmVmIuvVy&key=7393e0b0f5ed79b36c8ab7b2ab777c353af36ee651295c3ef9a089f946215a36d0880cc2c11caff523636180a4ec5265e85931f6bf0c6c6eb4d8cad56a2dfefd4eee9f5320591ffcac511ef82b7cd216&ascene=1&uin=MTM3NTg3NjEyMg%3D%3D&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=YFznKAudlcVM77Jt2jnYOOO%2BR4DZ2shcjmgoRp7KoC%2FvBzvRt0e2txe%2Fg7Q1S%2FJd)
