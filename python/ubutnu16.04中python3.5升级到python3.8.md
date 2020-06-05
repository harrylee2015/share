# ubuntu16.04中python3.5升级到python3.8

## 前言

ubuntu16.4中系统本身自带2.7x 和 3.5 版本的python，实质上python3.5已经时16年，17年发行的版本，距今有一段时间了，
在实际的开发过程中我们往往会遇到一些低版本的库不兼容的问题，需要升级一下本地开发环境的python版本

## 升级步骤

1. 下载源码包

    我这里选择的时python3.8版本
 
    [https://www.python.org/ftp/python/](https://www.python.org/ftp/python/)
  
  
2. 解压

 ```
  tar zxvf Python-3.8.0.tar.xz
  cd Python-3.8.0
 ``` 
3.编译（如果时普通用户的话，要给赋予sudo权限）

  ```
  ./configure
  make
  make install
 ``` 
 
4.添加软连接

  ```
  先找的python3.8解释器位置，一般是
  /usr/local/bin/python3.8
  删除原来的软连接
  rm -rf /usr/bin/python3
  rm -rf /usr/bin/pip3
  #添加python3的软链接
  ln -s /usr/local/bin/python3.8 /usr/bin/python3
  #添加 pip3 的软链接
  ln -s /usr/local/bin/pip3.8 /usr/bin/pip3
  
  ```

5. 检查更新后的python版本

  ```
  harry@harry-VirtualBox:~$ python3 --version
  Python 3.8.0
  ```
