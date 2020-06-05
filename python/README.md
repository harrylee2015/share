# python


## python官方文档

   [中文文档](https://docs.python.org/zh-cn/3.7/using/index.html)

## python的版本管理工具pyenv

   [pyenv项目地址](https://github.com/pyenv/pyenv)

## python的项目依赖管理工具pip使用

 每个python版本都有自带的pip,列举一些常用的操作命令
 一般在执行命令时都会制定python的版本，比如
 ```
 python3 -m pip install xxxx
 
 ```
 
 命令|作用
 ----|---
pip freeze > requirements.txt| 该方法仅可以使用在虚拟环境中，会将python 解释器下的所有包都导出
pip install pipreqs|安装pipreqs 包
pipreqs ./ --encoding=utf-8 --force | 表示覆盖该原有requirements.txt,只添加本项目有关的依赖

## python中使用protobuf序列化

 1. 安装protoc二进制文件 
 
    [ 二进制文件下载](https://github.com/protocolbuffers/protobuf/releases)
 
 2. 使用pip安装protobuf源码包
 
    ```
     python3 -m install protobuf
    ```
    
 3. 赋值注意事项
 
    **对象赋值，需要逐个字段赋值，对象之间不可以直接赋值**
 
    **protobuf中如何对象不赋值的话，默认就为空**，不需要特殊处理
    
    
 4. 序列化与反序列化  
 
    xxx.SerializeToString()   
    xxx.ParseFromString()

