# Ubuntu中通过SuperVisor添加守护进程

## 前言

 守护进程用于保持一个指定程序（dll）时刻保持运行。在命令行终端中通过chain33 -f chain33.toml 命令执行的程序，
 在退出命令行终端后，程序自动终止。添加守护进程后，即使终端退出，程序仍可后台执行。可用于执行定时任务。
 
 
## 安装&配置
 
1. 安装（环境：ubuntu 16.04 ）

```
sudo apt-get install supervisor  （必须在root下执行）
```
2. 修改配置文件

配置文件路径：/ect/supervisor/conf.d/

添加配置守护进程的配置文件
```
vim chain33.conf
```
增加：

1）添加被守护的程序
```
[program:chain33]
command=chain33 -f chain33.toml  ;要执行的指令 
directory=/root/lihailei        ;chain33的路径
autostart=true
autorestart=true
numprocs=1
process_name=chain33        ;名称utorestart = true
```
2)修改 supervisord.conf

开启查看进程的界面
```
[inet_http_server]
port=0.0.0.0:7002
```
3）重启
```
supervisorctl reload
```
4)访问localhost:7002（与配置文件中的端口相同）即可查看当前正在运行的所有程序

 

supervisord.conf 配置文件完整示例：

```
; supervisor config file

[unix_http_server]
file=/var/run/supervisor.sock ; (the path to the socket file)
chmod=0700 ; sockef file mode (default 0700)_

[inet_http_server]
port=127.0.0.1:7002

[supervisord]
logfile=/var/log/supervisor/supervisord.log ; (main log file;default $CWD/supervisord.log)
pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
childlogdir=/var/log/supervisor ; ('AUTO' child log dir, default $TEMP)

; the below section must remain in the config file for RPC
; (supervisorctl/web interface) to work, additional interfaces may be
; added by defining them in separate rpcinterface: sections
[rpcinterface:supervisor]
supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface

[supervisorctl]
serverurl=unix:///var/run/supervisor.sock ; use a unix:// URL for a unix socket

; The [include] section can just contain the "files" setting. This
; setting can list multiple files (separated by whitespace or
; newlines). It can also contain wildcards. The filenames are
; interpreted as relative to this file. Included files *cannot*
; include files themselves.

[include]
files = /etc/supervisor/conf.d/*.conf
```
 
