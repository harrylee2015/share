
## 关于linux一些其他知识点的补充

[linux中信号量](http://gityuan.com/2015/12/20/signal/)

## 本地时区切换

```
rm /etc/localtime

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

## 查找根目录下大于100M的文件

```
find / -type f -size +100000000c -exec du -sh {} \;
```

## linux中手动添加虚拟内存交换区

```
sudo fallocate -l 20G /mnt/.swapfile # 创建 20G 空文件
sudo mkswap /mnt/.swapfile    # 转换为交换分区文件
sudo chmod 600 /mnt/.swapfile # 修改权限为 600
sudo swapon /mnt/.swapfile    # 激活交换分区
free -m # 查看当前内存使用情况(包括交换分区)


##关闭交换分区
sudo swapoff /mnt/.swapfile
rm -rf /mnt/.swapfile
```


##查看端口占用：

```
lsof -i :port

```

##查看CPU负载：

```
top
```

## 查看内存占用：

```
free -m 
```
## 发送信号给进程：

```
kill -9 pid
使用命令「kill」来发送信号给进程，其中参数「-SIG」表示发送的信号，「pid」表示进程的ID。例如「kill -SIGTERM pid」表示发送终止信号给进程。
```
