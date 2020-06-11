# question

1. make 与 new 的区别
new(T) 返回 T 的指针 *T 并指向 T 的零值。
make(T) 返回的初始化的 T，只能用于 slice，map，channel。

2. golang中struct是否可以比较

     **看结构体本身是否含有不能比较的成员变量**
   
   行为|数据类型
   ---|---
   可排序|Integer,Floating-point,String
   可比较|Integer,Floating-point,String，Boolean,Complex，Pointer,Channel,Interface,Array
   不可比较|Slice,Map,Function
   
   同样struct是否可以作为map的key?
   答案是可以，也不可以，**struct必须是可比较的,才能作为key，否则编译时报错**
   
   


2. goroutine的调度机制
  
   [G,M,P模型](https://mp.weixin.qq.com/s/jFBorQ6fRopqmEK1voKGWA) 

3. golang内存模型

   [golang interview questions](https://www.cnblogs.com/yulibostu/articles/12056197.html)
