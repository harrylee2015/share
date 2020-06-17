# python


## python官方文档

   [中文文档](https://docs.python.org/zh-cn/3.7/using/index.html)

## python的版本管理工具pyenv

   [pyenv项目地址](https://github.com/pyenv/pyenv)
   
   * 安装pyenv
   
      ```
      
       git clone https://github.com/pyenv/pyenv.git ~/.pyenv
       
       echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
       
       echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
       
       echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
      
      ```
   
   
   * 利用pyenv工具安装python
   
       ```
          pyenv install 3.7.7
    
       ```
    
   * 下载加速
    
      ```
       vim /home/harry/.pyenv/plugins/python-build/share/python-build/3.7.7
       
       将原来的 https://www.python.org/ftp
       替换为  http://mirrors.sohu.com
       
      ```
      
      重新执行安装命令
      
      
   * 错误处理
    
      ```
       Last 10 log lines:
       **kwargs).stdout
       File "/tmp/python-build.20200617160756.12100/Python-3.7.7/Lib/subprocess.py", line 512, in run
       output=stdout, stderr=stderr)
         subprocess.CalledProcessError: Command '('lsb_release', '-a')' returned non-zero exit status          ...                                                                                            
         Traceback (most recent call last):
         File "/usr/bin/lsb_release", line 25, in <module>
       import lsb_release
          ModuleNotFoundError: No module named 'lsb_release'
   
      ```
   
     删除报错地方
     
     ```
     rm -rf /usr/bin/lsb_release
     ```
   
   * 设置全局环境
    
     ```
      pyenv global 3.7.7
     ```
   

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

