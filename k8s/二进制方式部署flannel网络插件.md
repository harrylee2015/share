# 二进制方式部署flannel网络插件

## 前言

在k8s集群中，不同节点中得容器能够互通主要归功于CNI插件，常见得CNI插件有flannel (隧道方式，类似于OSI七层网络模型，通过数据包得再封装得方式，flannel 使用 vxlan 技术为各节点创建一个可以互通的 Pod 网络，使用的端口为 UDP 8472（需要开放该端口，如公有云 AWS 等）。）
calico（路由方案），这里我们学着如何通过二进制得方式去安装flannel网络插件。
flanneld 第一次启动时，从 etcd 获取配置的 Pod 网段信息，为本节点分配一个未使用的地址段，然后创建 `flannedl.1` 网络接口（也可能是其它名称，如flannel1 等）。
flannel 将分配给自己的 Pod 网段信息写入 `/run/flannel/docker` 文件，docker 后续使用这个文件中的环境变量设置 `docker0` 网桥，从而从这个地址段为本节点的所有 Pod 容器分配 IP


## 前提准备
