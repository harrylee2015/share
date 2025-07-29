# vscode远程开发环境配置

## 1. 下载 [vscode 软件](https://code.visualstudio.com/)

## 2. 远程linux主机上面安装 golang开发环境

```
 wget https://dl.google.com/go/go1.24.5.linux-amd64.tar.gz

 sudo tar -C  /usr/local -xzf go1.24.5.linux-amd64.tar.gz

 # 写入一下环境变量,结束时source /etc/profile使环境变量生效

   # Go Environment Variables  /etc/profile
   export PATH=$PATH:/usr/local/go/bin
   export GOPATH=$HOME/go
   export PATH=$PATH:$GOPATH/bin
   export GOPROXY=https://goproxy.cn,direct
   
```

## 3.免密码登录配置

```
1. 生成密钥对（客户端和服务端都要生成），执行如下命令，一路enter件即可生成 .ssh文件夹下会有相应的rsa密钥对
 ssh-keygen -t rsa
 
2.将windows(客户端的公钥copy到 服务端（linux)上面
 ssh-copy-id  harry@192.168.2.186

3. vscode中安装remote-ssh插件
  3.1） 打开扩展面板
      在 VS Code 中，点击左侧活动栏的扩展图标（或使用快捷键 Ctrl+Shift+X）打开扩展面板。
  3.2） 搜索并安装插件
      在搜索框中输入 Remote-SSH，选择第一个插件（由 Microsoft 发布），然后点击“安装”按钮。
  3.3）配置 SSH 连接
   安装完成后，打开命令面板（快捷键 Ctrl+Shift+P），输入 Remote-SSH: Connect to Host，然后按照提示输入远程服务器的 SSH 地址（格式为 username@hostname）
   也可以在 ~/.ssh/config 文件中配置

        Host  harry
         HostName 192.168.2.186
         User root
         Port 22
         IdentityFile ~/.ssh/id_rsa
     

```

## 4.安装go相关依赖插件

```

1. 打开命令面板
首先，按下 Ctrl+Shift+P 打开 VS Code 的命令面板。这是 VS Code 中执行各种命令的快捷入口。

2. 输入插件安装命令
在命令面板中输入 Go: Install/Update Tools，然后选择该选项。这个命令会打开一个列表，显示所有可用的 Go 工具和插件。

3. 选择需要安装的依赖
在弹出的工具列表中，勾选需要安装的 Go 依赖插件。通常建议勾选以下常用工具：

gopls：Go 语言服务器，提供代码补全、跳转到定义等功能。
go-outline：用于代码大纲和结构化显示。
gofmt：用于代码格式化。
4. 确认安装
选择完成后，点击右下角的 OK 按钮，VS Code 会自动下载并安装所选的依赖插件。

5. 验证安装
安装完成后，可以通过重新打开 Go 文件并尝试使用功能（如代码跳转或自动补全）来验证插件是否正常工作。如果遇到问题，可以重启 VS Code 或重新加载窗口。

```
