# go module私服的搭建

## go module私服的安装
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

当外面机器需要用该私服下载一些依赖时，只需导入如下变量，执行下载命令
```bash
//开启 module功能
export GO111MODULE=on
//私服代理地址,用于下载相关依赖
export GOPROXY=http://xxxx:3000
```

## go module 功能的使用

我们这里以chain33中plugin项目为例，示范一下如果用module去打包一个项目

1.首先我们先在刚刚搭建的私服机器上面拉取原有代码，并且换成一个新的module分支

```bash
 略
```

2.接着初始化init go.mod
```bash
 cd plugin
 //由于原来的项目地下有vendor目录，mod会自动去copy转换，但是由于一些库的地址不能直接访问，所以肯定会失败，只会生成一个go.mod文件
 go mod init
 
 //会报如下错误信息
 go: creating new go.mod: module github.com/33cn/plugin
 go: copying requirements from vendor/vendor.json
 go: converting vendor/vendor.json: stat github.com/btcsuite/btcd/wire@: unknown revision 
 go: converting vendor/vendor.json: stat github.com/btcsuite/btcd/chaincfg@: unknown revision 
 go: converting vendor/vendor.json: stat github.com/btcsuite/btcd/chaincfg/chainhash@: unknown revision 
 go: converting vendor/vendor.json: stat github.com/btcsuite/btcd/rpcclient@: unknown revision 
 go: converting vendor/vendor.json: stat github.com/coreos/etcd/pkg/fileutil@: unknown revision 
 go: converting vendor/vendor.json: stat golang.org/x/time/rate@: unrecognized import path "golang.org/x/time/rate" (https fetch: Get   https://golang.org/x/time/rate?go-get=1: dial tcp 216.239.37.1:443: i/o timeout)
 go: converting vendor/vendor.json: stat golang.org/x/crypto/bcrypt@: unrecognized import path "golang.org/x/crypto/bcrypt" (https fetch: Get https://golang.org/x/crypto/bcrypt?go-get=1: dial tcp 216.239.37.1:443: i/o timeout)

 
```
3.修改go.mod文件，通过replace替换依赖包的地址

如下所示，一般访问不了的库，在github上面都有开源的地址，只需替换一下，增加地址重定向即可

```bash
replace (
        cloud.google.com/go => github.com/googleapis/google-cloud-go v0.43.1-0.20190808215159-84f66600e42d
        golang.org/x/crypto => github.com/golang/crypto v0.0.0-20190313024323-a1f597ede03a
        golang.org/x/exp => github.com/golang/exp v0.0.0-20190731235908-ec7cb31e5a56
        golang.org/x/image => github.com/golang/image v0.0.0-20190227222117-0694c2d4d067
        golang.org/x/lint => github.com/golang/lint v0.0.0-20190409202823-959b441ac422
        golang.org/x/mobile => github.com/golang/mobile v0.0.0-20190312151609-d3739f865fa6
        golang.org/x/mod => github.com/golang/mod v0.1.0
        golang.org/x/net => github.com/golang/net v0.0.0-20190318221613-d196dffd7c2b
        golang.org/x/oauth2 => github.com/golang/oauth2 v0.0.0-20190523182746-aaccbc9213b0
        golang.org/x/sync => github.com/golang/sync v0.0.0-20190227155943-e225da77a7e6
        golang.org/x/sys => github.com/golang/sys v0.0.0-20190318195719-6c81ef8f67ca
        golang.org/x/text => github.com/golang/text v0.3.0
        golang.org/x/time => github.com/golang/time v0.0.0-20190308202827-9d24e82272b4
        golang.org/x/tools => github.com/golang/tools v0.0.0-20190529010454-aa71c3f32488
        google.golang.org/api => github.com/googleapis/google-api-go-client v0.7.0
        google.golang.org/appengine => github.com/golang/appengine v1.6.1-0.20190515044707-311d3c5cf937
        google.golang.org/genproto => github.com/google/go-genproto v0.0.0-20190522204451-c2c4e71fbf69
        google.golang.org/grpc => github.com/grpc/grpc-go v1.21.0
)

```
重新执行打包命令,如还有问题，可根据提示信息进行处理

```bash
go mod tidy
```
