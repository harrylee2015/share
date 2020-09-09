# 1. golang中性能优化

<!-- TOC -->

- [1. golang中性能优化](#1-golang中性能优化)
  - [1.1. 前言](#11-前言)
  - [1.2. cpu耗时优化](#12-cpu耗时优化)
  - [1.3. 网络io性能优化](#13-网络io性能优化)
  - [1.4. struct字段顺序调整优化内存分配](#14-struct字段顺序调整优化内存分配)
  - [1.5. string与[]byte相互转换](#15-string与byte相互转换)
  - [1.6. 关于map使用注意事项](#16-关于map使用注意事项)
  - [1.7. 了解defer](#17-了解defer)
  - [1.8. 字符串拼接](#18-字符串拼接)
  - [1.9. reflect的性能影响](#19-reflect的性能影响)
  - [1.10. channel并不是很高效](#110-channel并不是很高效)
  - [1.11. 参考资料](#111-参考资料)

<!-- /TOC -->

## 1.1. 前言

  这里总结一下golang项目中如何进行的性能优化

  **调优的基本思路**
  1. 对外接口协议不能改变
  2. 了解需求和代码演进过程
  3. 确定资源消耗类型
  4.  控制运算数据输入量
  5.  提高 CPU 利用率
  6.  提高缓存命中率 

## 1.2. cpu耗时优化

   1. make时提前预估size
   2. 临时的map、slice采用sync.Pool
   3. 大于32Kb也可用sync.Pool
   4. 不滥用goroutine，减少gc压力
   5. 不滥用mutex，减少上下文切换
   6. []byte与string临时变量转换用unsafe
   7. 减少reflect、defer使用
   8. atomic无锁使用

## 1.3. 网络io性能优化

   1. 批量接口支持
   2. http 长连接
   3. redis pipeline
   4. db、redis连接池
   5. 增加缓存,提高命中率
   6. 大量数据压缩传输
 
## 1.4. struct字段顺序调整优化内存分配

  **golang中的字节对齐规则**

  1. 数据成员对齐规则： 结构体(struct) 或者 联合体(union)的数据成员第一个数据成员放在offset为0的地方，以后每个数据成员的地址必须是它本身大小或者对齐参数两者中较小的一个的倍数。
  2. 整体对齐规则： 在数据成员完成各自对齐后，结构体或者联合体本身也要进行对齐，整体长度必须是对齐参数和结构体最长元素长度中较小的一个的倍数。
  3. 默认对齐参数和机器有关，64位平台的默认对齐参数是8, 32位的是4
   
   参数规则分析一下下面结构体的数据分配

   ```
   // 64位机器上,align = 8
     type People struct {
         age uint8        // offset =0, 自身占用1个字节 
         sex bool         // offset =1, 自身占用1个字节
         salary uint64    //offset 需要偏移量6， offset =8, 自身占用8个字节
         name string      // string类型大小是16字节，根据规则1,min(align,unsafe.Sizeof(name)), offset=16,name自身占用16字节
     }
     //整体对齐，整体长度是1+1+6+8+16=32字节，已经是min(align，unsafe.Sizeof(name))最小数，8的倍数了，所以整个People在内存中共需要 32字节的大小

     调整一下字段顺序

     type People2 struct {
         age uint8      // setoff=0,age自身占用1字节
         salary unint64  // setoff=8,salary自身占用 8个字节
         name string       // offset =16, 自身占用16个字节 
         sex bool     //offset=32,自身占用 1个字节
     }
     // 整体对齐，整体长度是 33字节，根据整体对齐规则，需要再填充7个字节才满足 8*5=33+7,因此 最终，People2在内存中整体大小应该是40字节

   ```
  
  从上面两个例子可以得出，适当调整struct中字段顺序，减少内存分配，提高内存使用率

## 1.5. string与[]byte相互转换

   我们经常会遇到string与[]byte的相互转换，但是这种转换是有代价的，string与[]byte并不共享底层内存空间，所以每次转换都伴随着内存的分配与底层字节的拷贝。
   我们可以借助unsafe完成指针类型转换,避开内存分配与复制，从而提升性能。

```
 /*
  struct string{
    uint8 *str;
    int len;
  }
  struct []uint8{
    uint8 *array;
    int len;
    int cap;
  }
  uintptr是golang的内置类型，是能存储指针的整型，uintptr的底层类型是int，它和unsafe.Pointer可相互转换。
  但是转换后的string与[]byte共享底层空间，如果修改了[]byte那么string的值也会改变，就违背了string应该是只读的规范了，可能会造成难以预期的影响。
*/
func str2byte(s string) []byte {
	x := (*[2]uintptr)(unsafe.Pointer(&s))
	b := [3]uintptr{x[0],x[1],x[1]}
	return *(*[]byte)(unsafe.Pointer(&b))
}
func byte2str(b []byte) string{
	return *(*string)(unsafe.Pointer(&b))
}
```
这个案例可以我的harry_tool项目中找到它的基准测试用例，感兴趣的同学可以去试试

## 1.6. 关于map使用注意事项

 **预设容量**

   map可以动态扩容，所以我们可以不关心map的大小，但是每次动态扩容时需要付出数据拷贝和重新哈希成本，如果我们能预先知道一个map最终的容量，那么最好在初始化时就指定。
   
   ```
   bigMap := make(map[int]int,100000) 
   ```
 **直接存储小对象值而不是指针**

  对于小对象，直接将数据交由 map 保存，远比用指针高效。这不但减少了堆内存分配，关键还在于垃圾回收器不会扫描非指针类型 key/value 对象。

```
//存储值对象 
m := make(map[int]int,1000)
for i := 0 ;i<10000;i++ {
    m[i]=i;
}

//存储指针对象
//如果value是个小对象,直接存储值会比较好
m := make(map[int]*int,1000)
for i := 0 ;i<10000;i++ {
    value := i 
    m[i]=&value;
}
```
**手动删除没有元素的map**

map可以动态扩容，我们可以不断的往map中添加新元素,但是map并不会自动收缩空间，即使一个map中的所有元素都被删除，map依然会保留所有已分配的空间。

```
var dict map[int]int = make(map[int]int)
for i := 0 ;i<100000;i++ {
    dict[i] = i 
}
for key := range dict {
    delete(key)  //即使删掉所有的元素，dict的容量仍然>=100000
}
// 如果不再使用dict那么手动设置为nil 
// dict=nil 
//也可以把dict指向一个新创建的小map，原有的map所占用的内存空间会被回收
//dict = make(map[int]int) 
``` 

## 1.7. 了解defer

defer是提高可读性和避免资源未释放的非常有用的关键字，是使用golang写出可靠、稳定程序的利器。

defer后面的表达式会被放入一个类似于栈(stack)的结构，在当前方法返回的时候，列表中的表达式会按照后进先出的顺序执行。

defer本身会有一定的性能损失，但是和它带来的好处相比根本不值得一提,我们需要注意的关键点是defer表达式会在函数返回时被调用,意味着有些资源只能在函数结束时才被释放。

    func f(){
        m.lock()
        defer m.unlock()
        //....业务处理逻辑
        //这是很常见的上锁方式，但是m.unlock()只会在函数返回时调用，如果业务处理逻辑耗时很长,那么会一直占用着锁，在高并发情况下严重影响性能。
        //解决办法是找到**最小临界区**，在处理完最小临界区后及时释放掉锁。
    }
    func f() {
        m.lock()
        //...最小临界区
        m.unlock()
        //...继续处理
    }


## 1.8. 字符串拼接

字符串的拼接大概有以下几种方式

**fmt.Sprintf**

```
fmt.Sprintf(“%s%s%d%s%s”,”hello”,”world”,2016,”come”,”on”)
//这种方式效率最低，但是代码最简单，最优雅
```

**使用”+”**

 使用”+”拼接字符串 "hello"+"world"+ strconv.FormatInt(2016,10) +"come"+"on"//比fmt.Sprintf()高效一些，但是代码很难看

**使用strings.Join()**

将参数组装成[]string，然后调用strings.join,效率最高的一种方式，推荐使用

```
strs := []string{"hello","world",strconv.FormatInt(2016,10),"come","on"}
str = strings.Join(strs,"")
```
为什么使用string.Join效率最好呢,来看下strings.Join的代码
```
func Join(a []string, sep string) string {
    //计算最终字符串的长度，根据最终长度创建[]byte,避免拼接过程中内存重新分配
    n := len(sep) * (len(a) - 1)
    for i := 0; i < len(a); i++ {
        n += len(a[i])
    }
    b := make([]byte, n)
    //使用copy函数是最高效的
    bp := copy(b, a[0])
    for _, s := range a[1:] {
        bp += copy(b[bp:], sep)
        bp += copy(b[bp:], s)
    }
    return string(b)
}
```

但是有时候很难将参数拼接成[]string，这时我们可以使用byte.Buffer

    var buffer bytes.Buffer
    for i := 0; i < 1000; i++ {
        buffer.WriteString("a")
    }

## 1.9. reflect的性能影响

反射带来了极大的方便，但是同时也有一定的性能损失。性能要求极高的模块中应该注意反射所带来的性能损失。
JSON是一种常用的数据交换格式，但Go的encoding/json库依赖于反射来对json进行序列化和反序列化。使用[ffjson](https://github.com/pquerna/ffjson)，可以通过使用代码生成的方式来避免反射的使用，相比使用原生库可以提升2~3倍的性能。

## 1.10. channel并不是很高效

使用golang语言编程有一条很重要的思想是Don't communicate by sharing memory, share memory by communicating.而channel则是不同goroutine之间communicate的通道。但是channel的底层实现也不是无锁的，往channel中读写数据都是需要加锁的，而且锁的力度还很大。
在一些情况下可以使用利用atomic实现无锁的结构([ring buffer](https://github.com/Workiva/go-datastructures/blob/master/queue/ring.go))来替代channel以提高程序的性能。
而在有些情况下使用sync.Mutex、atomic不仅比使用channel效率更高代码还更简洁明了。Use whichever is most expressive and most simple.


## 1.11. 参考资料

 [golang高性能编程技巧](https://studygolang.com/articles/6859?fr=email)
 