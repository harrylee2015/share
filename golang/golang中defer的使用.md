# golang中defer的使用

## 前言

  defer是Go语言提供的一种用于注册延迟调用的机制，每一次defer都会把函数压入栈中，当前函数返回前再把延迟函数取出并执行。

defer语句并不会马上执行，而是会进入一个栈，函数 return 前，会按先进后出（FILO）的顺序执行。也就是说最先被定义的defer语句最后执行。先进后出的原因是后面定义的函数可能会依赖前面的资源，自然要先执行；否则，如果前面先执行，那后面函数的依赖就没有了。



## 踩坑点


使用 defer 最容易采坑的地方是和**带命名返回参数**的函数一起使用时

```
defer语句定义时，对外部变量的引用是有两种方式的，分别是作为函数参数和作为闭包引用。
作为函数参数，则在 defer定义时就把值传递给defer，并被缓存起来；作为闭包引用的话，则会在 defer 函数真正调用时根据整个上下文确定当前的值。
```




避免掉坑的关键是要理解这条语句：

```
return xxx

```
这条语句并不是一个原子指令，经过编译之后，变成了三条指令：

```
1. 返回值 = xxx
2. 调用 defer 函数
3. 空的 return
```

详细可以阅读seekload老师写的这篇文章[defer不知道的细节](https://mp.weixin.qq.com/s?__biz=MzI2MDA1MTcxMg==&mid=2648466918&idx=2&sn=151a8135f22563b7b97bf01ff480497b&chksm=f2474389c530ca9f3dc2ae1124e4e5ed3db4c45096924265bccfcb8908a829b9207b0dd26047&mpshare=1&scene=1&srcid=1227gCWEHLJz4UGpHTosBgPG&sharer_sharetime=1577409752758&sharer_shareid=2b86561e1950790af184ab7b506f6455&key=9178b5e3f94ce552d995a39049695892eaf484f77eccfb5aad9cfa4415d5197b2b1578f7a1279d21ddfdc921262090214106a58e72a03792c45cdb3aac0f878e9055a2b13ca2b0c7edf605cdb9b703f4&ascene=1&uin=MTM3NTg3NjEyMg%3D%3D&devicetype=Windows+10&version=62070158&lang=zh_CN&exportkey=AUHQk9pxIL9WAR7TqBlWDa0%3D&pass_ticket=COL2SNRx%2FsDIDp%2Fqzbj0AoD%2BkrjdOviaLhe2s8RXH1aZeAHrtZmEkwFrfpiAcyrK)，案例结合，写的非常棒

