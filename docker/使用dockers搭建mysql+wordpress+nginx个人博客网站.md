# 使用dockers搭建mysql+wordpress+nginx个人博客网站

## 1.创建自定义网络

```
harry@harry-VirtualBox:/apps/wordpress$ docker network create -d bridge --subnet 10.0.10.0/24 appsnet
de2bbc1bed177924c624c806c7c613fcf68cafb8d7da2ac7e924af954a34917c
```
 查看网络信息
 
 ```
 harry@harry-VirtualBox:/apps/wordpress$ ifconfig
....

br-7871a7d01f53 Link encap:以太网  硬件地址 02:42:0b:f7:b2:aa  
          inet 地址:172.18.0.1  广播:172.18.255.255  掩码:255.255.0.0
          inet6 地址: fe80::42:bff:fef7:b2aa/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  跃点数:1
          接收数据包:2 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:52 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:0 
          接收字节:56 (56.0 B)  发送字节:6132 (6.1 KB)

br-de2bbc1bed17 Link encap:以太网  硬件地址 02:42:b0:23:ca:4c  
          inet 地址:10.0.10.1  广播:10.0.10.255  掩码:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  跃点数:1
          接收数据包:0 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:0 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:0 
          接收字节:0 (0.0 B)  发送字节:0 (0.0 B)

docker0   Link encap:以太网  硬件地址 02:42:5d:40:15:02  
          inet 地址:172.17.0.1  广播:172.17.255.255  掩码:255.255.0.0
          inet6 地址: fe80::42:5dff:fe40:1502/64 Scope:Link
          UP BROADCAST MULTICAST  MTU:1500  跃点数:1
          接收数据包:0 错误:0 丢弃:0 过载:0 帧数:0
          发送数据包:3 错误:0 丢弃:0 过载:0 载波:0
          碰撞:0 发送队列长度:0 
          接收字节:0 (0.0 B)  发送字节:266 (266.0 B)
....
 ```
 
## 2. 初始化数据库(使用镜像：mysql:5.7)

```
harry@harry-VirtualBox:/apps/wordpress$ docker run --name db -d --net appsnet -p 127.0.0.1:3306:3306  -v $(pwd)/db/data:/var/lib/mysql --env-file db/env.list  mysql:5.7
ddf30a6339e092f70991f77bef195d5c0a4fbb227a23dcf99fefcd8547cda4c5
```
数据库配置项
```
harry@harry-VirtualBox:/apps/wordpress$ cat db/env.list 
MYSQL_DATABASE=wpdb
MYSQL_USER=wpuser
MYSQL_PASSWORD=wppass@magedu.Com
MYSQL_RANDOM_ROOT_PASSWORD=1
```
 查看容器网络

```
harry@harry-VirtualBox:/apps/wordpress$ docker inspect db
{
...
 "Gateway": "10.0.10.1",
 "IPAddress": "10.0.10.2",
...

}
```

## 3. 运行WordPress容器(使用镜像：wordpress:php7.2-fpm)
修改配置文件
```
harry@harry-VirtualBox:/apps/wordpress$ cat wp/env.list 
WORDPRESS_DB_HOST=db.magedu.com
WORDPRESS_DB_USER=wpuser
WORDPRESS_DB_PASSWORD=wppass@magedu.Com
WORDPRESS_DB_NAME=wpdb
```
启动wordpress容器
```
harry@harry-VirtualBox:/apps/wordpress$ docker run --name wp --net appsnet -d --env-file wp/env.list -v $(pwd)/wp/fpm/www.conf:/usr/local/etc/php-fpm.d/www.conf:ro --add-host "db.magedu.com:10.0.10.2" wordpress:php7.2-fpm
9c34d09c2aee4fe0dc1fe84bd7f7de7a052f94f3b15eae4f463696e06489251f
```
## 4. 配置和运行nginx，代理至wp容器(使用镜像：nginx:alpine)

修改nginx默认配置文件 default.conf

```
harry@harry-VirtualBox:/apps/wordpress$ cat nginx/default.conf

upstream wpserver {
  #请求分发路径
	server wp.magedu.com:9000;
}
server {
    listen       80;
    server_name  localhost;
    location / {
        root   /var/www/html;
        index  index.php index.html index.htm;
        try_files $uri $uri/ /index.php?$args;
    }

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
        # 此路径为php-fpm服务器上*.php页面文件存放路径，可以docker image inspect wordpress:php7.2-fpm 查看 WorkingDir为"/var/www/html"
        root           /var/www/html;
        fastcgi_pass   wpserver;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

}
```

启动nginx做反向代理

```
harry@harry-VirtualBox:/apps/wordpress$ docker run --name nginx -d --net appsnet -p 80:80 -v $(pwd)/nginx/default.conf:/etc/nginx/conf.d/default.conf:ro --add-host "wp.magedu.com:10.0.10.3" nginx:alpine
750df31203e45f09fb5949023fde53686d29f0071787641b94c04acaa1f4582c
harry@harry-VirtualBox:/apps/wordpress$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                                 NAMES
750df31203e4        nginx:alpine           "nginx -g 'daemon of…"   6 seconds ago       Up 5 seconds        0.0.0.0:80->80/tcp                    nginx
9c34d09c2aee        wordpress:php7.2-fpm   "docker-entrypoint.s…"   About an hour ago   Up About an hour    9000/tcp                              wp
ddf30a6339e0        mysql:5.7              "docker-entrypoint.s…"   2 hours ago         Up 2 hours          127.0.0.1:3306->3306/tcp, 33060/tcp   db

```
访问宿主机80端口即可
