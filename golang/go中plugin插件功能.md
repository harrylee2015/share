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
