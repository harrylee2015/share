# go moudle私服的搭建

## 私服的安装
首先你需要安装 Go1.11或者以上版本

用的是谷歌开源的项目[Athens](https://github.com/gomods/athens)，雅典娜。
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
export GOPROXY=http://xxxx:30000
```
