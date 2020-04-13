# 二进制方式部署k8s集群


## 角色规划

  节点名|ip |组件
  ----|----|----
  k8s-master|192.168.0.112|etcd,flannel,api-server,controllmanager,scheduler,kube-proxy
  node01|192.168.0.121|etcd,flannel,kube-proxy,kubelet
  node02|192.168.0.149|etcd,flannel,kube-proxy,kubelet

## 在Master节点部署组件

 **在执行这部操作之前需要要安装好etcd,docker,flannel插件，确保这些组件是正常工作的**
 
### 生成证书

 

 
 
