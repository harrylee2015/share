# go moudle私服的搭建

## 私服的安装
首先你需要安装 Go1.11或者以上版本
```bash
wget https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz

tar -xvf go1.12.7.linux-amd64.tar.gz

//创建gopath路径，后面module管理的所有版本依赖都会在gopath/pkg 目录下面
mkdir gopath
```
在/etc/profile中加入环境变量
```bash
export GOROOT=/root/go
export GOPATH=/root/gopath
export PATH=.:$PATH:$GOROOT/bin:$GOPATH/bin
#如下两个参数可配，可不配，因为我这里全部使用module模式，所以全加入到环境配置变量中去了
export GOPROXY=https://goproxy.io
export export GO111MODULE=on
```
配置完之后保存,执行如下命令是变量生效
```bash
source /etc/profile
```

私服是用的是谷歌开源的项目[Athens](https://github.com/gomods/athens)搭建的，雅典娜。

克隆下载项目代码
```bash
git clone https://github.com/gomods/athens
```
开启go module功能，并编译
```bash
//开启 module功能
export GO111MODULE=on
//公共代理地址,用于下载相关依赖
export GOPROXY=https://goproxy.io
cd athens/cmd/proxy
go install
//或者手动编译
go build -o proxy
```
编译完后会生成proxy二进制文件
执行启动命令即可
```bash
nohup ./autens >console.log 2>&1 &
```
私服就搭建好了，注意需要放行默认的3000端口
当外面机器需要用该私服下载一些依赖时
只需导入如下变量，执行下载命令
```bash
//开启 module功能
export GO111MODULE=on
//私服代理地址,用于下载相关依赖
export GOPROXY=http://xxxx:3000
```
