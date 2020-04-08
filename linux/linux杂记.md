
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
