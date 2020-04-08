# kubernetes学习


## k8s网络模型
 
- 一个pod一个IP

- 每个pod独立IP，pod内所有容器共享同一个IP

- 所有容器都可以与所有其他容器通讯

- 所有节点都可以与所有容器通讯



## kubernetes sheduler -preselect预选规则

 - NodiskConflict
 
 - CheckNodeMemoryPressure
 
 - NodeSelector 节点选择
 
 - FitResource 资源过滤
 
 - Affinity 亲和性
 
## kubernetes sheduler -optimize-select 最优选择机制

- SelectorSpreadPriority 传播选择  

- LeastRequestedPriority  空闲容量选择

- AffinityPriority 亲和性
