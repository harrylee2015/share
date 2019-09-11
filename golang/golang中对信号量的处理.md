## 前言
信号量用法很多，一般是涉及一下linux 相关信号的监听,应用场景一般用于程序的中断退出，或者通过user1用户自定义信号来启动配置文件的热更新

## 接口定义

```
//忽视指定的信号
func Ignore(sig ...os.Signal)
//检测某个信号是否被忽视
func Ignored(sig os.Signal) bool
//监听
func Notify(c chan<- os.Signal, sig ...os.Signal)
func Reset(sig ...os.Signal)
func Stop(c chan<- os.Signal)
```

### 应用案例
```
package main

import (
	"fmt"
	"os"
	"os/signal"
)

func main() {
	// Set up channel on which to send signal notifications.
	// We must use a buffered channel or risk missing the signal
	// if we're not ready to receive when the signal is sent.
	c := make(chan os.Signal, 1)
	signal.Notify(c, os.Interrupt)

	// Block until a signal is received.
	s := <-c
	fmt.Println("Got signal:", s)
}

```


**[golang中对信号量的处理](https://www.jianshu.com/p/ae72ad58ecb6)**
