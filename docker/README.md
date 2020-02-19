# Docker 

## 前言

 Docker 是一个开源的应用容器引擎，基于 Go 语言 并遵从 Apache2.0 协议开源。
 开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的Linux机器或Windows 机器上,也可以实现虚拟化。
 是一种新的云容器虚拟化技术。
 
 ## Docker的应用场景
 
 * Web 应用的自动化打包和发布。

 * 自动化测试和持续集成、发布。

 * 在服务型环境中部署和调整数据库或其他的后台应用。

 * 从头编译或者扩展现有的 OpenShift 或 Cloud Foundry 平台来搭建自己的 PaaS 环境。
 
 
 ## Docker学习资料
 
 * [Docker教程](https://www.runoob.com/docker/docker-tutorial.html)
 
 * [Docker官网](https://www.docker.com) 

 * [Github Docker源码](https://github.com/docker/docker-ce)
 
 ## Docker中的网络模型
 
 * bridge 网桥模式
 
    在网络方面，桥接网络是在网段之间转发流量的链路层设备。桥可以是在主机内核中运行的硬件设备或软件设备。

就Docker而言，网桥网络使用软件网桥，该软件网桥允许连接到同一网桥网络的容器进行通信，同时与未连接到该网桥网络的容器隔离。Docker网桥驱动程序会自动在主机中安装规则，以使不同网桥网络上的容器无法直接相互通信。

桥接网络适用于在同一 Docker守护程序主机上运行的容器。为了在不同Docker守护程序主机上运行的容器之间进行通信，您可以在OS级别管理路由，也可以使用覆盖网络。

启动Docker时，会自动创建一个默认的桥接网络（也称为bridge），除非另有说明，否则新启动的容器将连接到它。您还可以创建用户定义的自定义网桥网络。用户定义的网桥网络优于默认bridge 网络。

 * host 主机模式
 
 * overlay 叠加模式
 
 * macvlan 物理vlan模式
