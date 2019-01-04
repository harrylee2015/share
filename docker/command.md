# docker 命令小结

* 删除所有容器
```bash
sudo docker ps -a |grep -v IMAGE | awk '{print $1}' | xargs sudo docker rm
```
