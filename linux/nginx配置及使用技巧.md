# nginx配置使用技巧

## 指定配置文件启动方式

```
  sudo nginx  -c /mnt/data/nginx/conf/nginx.conf
```
## 柔和加载更新配置文件

```
  sudo nginx -s reload -c /mnt/data/nginx/conf/nginx.conf
```

## docker方式启动并指定配置文件方式

```
sudo docker run -d -p  8080:80 --name nginx-custom -v /mnt/data/nginx/conf/nginx.conf:/etc/nginx/nginx.conf nginx

```
