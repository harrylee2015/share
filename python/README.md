# python

<!-- TOC -->

- [python](#python)
  - [python官方文档](#python官方文档)
  - [python的版本管理工具pyenv](#python的版本管理工具pyenv)
  - [python的项目依赖管理工具pip使用](#python的项目依赖管理工具pip使用)
  - [python中使用protobuf序列化](#python中使用protobuf序列化)
  - [python中class实列化对象如何序列化json对象](#python中class实列化对象如何序列化json对象)
  - [python中如何使用setup工具在 pypi上发布自己的库](#python中如何使用setup工具在-pypi上发布自己的库)

<!-- /TOC -->
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

     sudo apt-get update
     sudo apt-get upgrade
     sudo apt-get dist-upgrade
     sudo apt-get install build-essential python-dev python-setuptools python-pip python-smbus
     sudo apt-get install build-essential libncursesw5-dev libgdbm-dev libc6-dev
     sudo apt-get install zlib1g-dev libsqlite3-dev tk-dev
     sudo apt-get install libssl-dev openssl
     sudo apt-get install libffi-dev
     ```
     
     重新安装
     
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

## python中class实列化对象如何序列化json对象

  * 自定义json序列化函数
  
  * 利用class中的__dict__字典属性，作为默认的匿名序列化函数
  
  下面一起看一个用例，分别展示通过这两种方式去实现class类的json序列化
  
  ```
  # 账户信息
class Account(object):
    def __init__(self, currency: int, balance: int, frozen: int, addr: str):
        self.currency = currency
        self.balance = balance
        self.frozen = frozen
        self.addr = addr

    # 自定义json序列化函数
    def obj_json(self, obj) -> object:
        return {
            "currency": obj.currency,
            "balance": obj.balance,
            "frozen": obj.frozen,
            "addr": obj.addr
        }


# 自定义反序函数
def jsonToClass(obj: json) -> Account:
    return Account(obj['currency'], obj['balance'], obj['frozen'], obj['addr'])



if __name__ == '__main__':
    account = Account(currency=0, balance=1, frozen=2, addr="")
    json_str = json.dumps(account, default=account.obj_json)
    print(json_str)
    # 通过__dict__属性实现序列化
    json_str = json.dumps(account, default=lambda account: account.__dict__)
    print(json_str)
    # 将json对象实例化成一个类对象
    acc = json.loads(json_str, object_hook=jsonToClass)
    print(type(acc))
    print(acc.balance)
    
  ```




## python中如何使用setup工具在 [pypi](https://pypi.org/)上发布自己的库

  * [官方发布指导](https://packaging.python.org/tutorials/packaging-projects/)
  
  
  **这里简单总结一下流程**
  
  1. 在[pypi](https://pypi.org/)注册账户
  
  2. 安装pip 工具
  
   ```
   python3 -m pip install  --upgrade setuptools wheel
   ```
  
  3. 构建项目setup.py文件，注意参考官方文档中目录路径
    
  4. 生成源码安装包
   
   ```
   python3 setup.py sdist bdist_wheel
   ```
   
  5. 注册项目
   
   ```
   python3 setup.py register
   ```
   
  6. 上传发布
  
   ```
   python3 setup.py sdist upload
   
   ```
   
## poetry 管理 python 项目
  
  [poetry](https://python-poetry.org/docs/#installing-with-the-official-installer)



## Windows 12 中安装 Python 3 的过程与其他 Windows 版本类似。您可以按照以下步骤进行操作：

1. 下载 Python 安装程序：

`前往 Python 官方网站 (https://www.python.org/downloads/)，下载最新的 Python 3 版本的安装程序。在网站上，您会看到各种版本的 Python 3，选择适合您系统的版本（32位或64位）。`

2. 运行安装程序：

`双击下载的 Python 安装程序，打开安装向导。确保选中 "Add Python X.X to PATH"（将 Python 添加到系统环境变量中），这样您就可以在命令行中直接运行 Python。然后点击 "Install Now"（立即安装）。`

3. 等待安装完成：

`安装程序会自动安装 Python，并添加到系统的 PATH 环境变量中。等待安装程序完成安装过程。`

4. 验证安装：

`打开命令提示符 (cmd)，输入 python --version，按下 Enter 键。如果安装成功，会显示 Python 的版本信息。

5. 安装完成：

至此，Python 3 已成功安装在您的 Windows 12 系统上。您可以开始使用 Python 编程了。

* 请注意，如果您需要在 Windows 中安装 Python 的特定版本，可以在 Python 官方网站的下载页面中选择该版本。另外，如果您需要在 Windows 中同时安装多个 Python 版本，可以使用 Python 的虚拟环境工具（如 virtualenv 或 venv）来管理不同版本的 Python。



## 在 Windows 12 中安装 pip 的步骤与其他 Windows 版本基本相同。以下是安装 pip 的常规步骤：

1. 下载 get-pip.py 脚本：

打开浏览器，前往 https://bootstrap.pypa.io/get-pip.py 右键点击页面上的 "Save link as"（另存为），保存 get-pip.py 脚本到您的计算机上。

2. 打开命令提示符：

在 Windows 12 中，按下 Windows键 + R 打开运行对话框，输入 cmd，然后按下 Enter 键，以打开命令提示符。

3. 运行 get-pip.py 脚本：

在命令提示符中，切换到保存 get-pip.py 脚本的目录。然后输入以下命令并按 Enter 键运行：

4. python get-pip.py

这将运行 get-pip.py 脚本，并自动下载安装 pip。

5. 等待安装完成：

安装过程会自动下载并安装 pip，等待安装完成。

6. 验证安装：

安装完成后，在命令提示符中输入以下命令并按 Enter 键验证 pip 是否成功安装：

pip --version

如果成功安装，会显示 pip 的版本信息。

至此，pip 已成功安装在您的 Windows 12 系统上，您可以使用 pip 来管理 Python 的包和库了。



