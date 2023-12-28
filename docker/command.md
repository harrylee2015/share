# docker 命令小结

* 删除所有容器
```bash
sudo docker ps -a |grep -v IMAGE | awk '{print $1}' | xargs sudo docker rm
```

* 查看docker 系统占用空间大小

```
sudo docker system df 
```

* 释放builder 缓存空间

```
sudo docker builder prune
```
