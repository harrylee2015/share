# python

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

