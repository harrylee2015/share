## 前言
go1.8之后开始支持plugin,动态加载插件的功能，之前也没有尝试，今天看了一下官方文档，结合官方给的demo,做一个梳理

## plugin库中接口定义

```bash
//加载的插件
type Plugin
// 调用Open加载插件，支持多协程方式的安全加载
    func Open(path string) (*Plugin, error)
// 调用Lookup接口查看symName的元素是否存在    
    func (p *Plugin) Lookup(symName string) (Symbol, error)
//插件中的某个原素，可以是变量也可以是方法    
type Symbol
```
## plugin插件的编写及编译
```bash
//demo

package main

import "fmt"

var V int

func F() { fmt.Printf("Hello, number %d\n", V) }



//设置编译模式为plugin
go build -buildmode=plugin
```
编译成*.so文件
```bash
root@instance-yzpr5rtu:~/hello_plugin# go build -buildmode=plugin -o hello_plugin.so hello_plugin.go 
root@instance-yzpr5rtu:~/hello_plugin# ll
total 3716
drwxr-xr-x  2 root root    4096 Aug 18 11:55 ./
drwx------ 10 root root    4096 Aug 18 11:53 ../
-rw-r--r--  1 root root      88 Aug 18 11:50 hello_plugin.go
-rw-r--r--  1 root root 3790256 Aug 18 11:55 hello_plugin.so
root@instance-yzpr5rtu:~/hello_plugin# 
```
## 调用插件

```bash
...
import "plugin"

func main() {
	p, err := plugin.Open("hello_plugin.so")
	if err != nil {
		panic(err)
	}
	v, err := p.Lookup("V")
	if err != nil {
		panic(err)
	}
	f, err := p.Lookup("F")
	if err != nil {
		panic(err)
	}
	*v.(*int) = 7
	f.(func())() // prints "Hello, number 7"
}

```
运行结果
```bash
root@instance-yzpr5rtu:~/hello_plugin# go run main.go 
Hello, number 7

```
由此我们可以看到调用plugin的过程其实就是一个类型推断的过程

## 参考资料
[plugin官方文档](https://golang.google.cn/pkg/plugin/)
