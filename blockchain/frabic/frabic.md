# 初始frabic
看了frabic官方使用文档，这里做个大概的总结

[初识frabic](https://github.com/harrylee2015/share/blob/master/frabic/%E5%88%9D%E5%A7%8Bfrabic.md)






# frabic 代码走读



***
模块名称 |所实现的主要功能
---------|--------------
bccsp|bccsp模块，blockchain cryptographic service provider，区块链加密服务提供者，提供各类型加密算法服务
cmd|cmd模块，系统入口函数，包括cli 都在这个模块
common|common模块，提供一些公共库
core|core模块，核心代码走在这里
devenv|devenv模块，提供开发环境
discovery|discovery模块，提供服务发现
docs|docs模块，相关技术文档，静态资源文件，默认官网加载的就是这里面的资源文件
examples|examples模块，提供cluster使用相关配置，及plugin二次开发样例
gossip|gossip模块，流言传播模块，实现订阅拉取指定块的数据
gradle|gradle模块，jar包
idemix|idemix模块，Identity Mixer Credential 身份混合凭据/证书
images|images模块，相关模块的docker镜像制作
integration|integration模块，集成测试
msp|msp模块，membership service provider,会员服务提供者
orderer|orderer模块，选择何种共识目前有solo,etcdraft,kafka等，开发者在这里可以实现自己的共识算法
peer|peer模块,节点
identity|identity模块，身份鉴权

***
















