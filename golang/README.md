* 关于golang的版本管理问题

      有时候我们会遇到有些项目是老的vendoer管理依赖包，但是此时我们的编译器都已经升级到1.13以上了，这个时候该怎么解决呢？
      这时我们只需要设置两个临时变量即可
      
      export GO15VENDOREXPERIMENT=1  #开启vendor版本管理
      export GO111MODULE="off"       #关闭新版本的go mod管理

* [golang 拉取私有仓库项目代码](https://andblog.cn/3002)
