
### 一、检查网络连接
1. **测试网络连通性**  
   执行以下命令，确认是否能正常访问 Docker Hub：
   ```bash
   curl -v https://registry-1.docker.io/v2/
   ```
   如果无法访问或响应缓慢，说明网络存在问题。
2. **使用 DNS 优化**  
   可以尝试修改 DNS 为公共 DNS，例如：
   ```bash
   sudo echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf > /dev/null
   ```
---
### 二、配置 Docker 镜像加速器
Docker Hub 在国内访问较慢，建议配置镜像加速器。以下是配置方法：
1. **编辑 Docker 配置文件**  
   打开或创建 `/etc/docker/daemon.json`，添加如下内容：
   ```json
   {
     "registry-mirrors": [
       "https://docker.mirrors.ustc.edu.cn",
       "https://hub-mirror.c.163.com",
       "https://mirror.baidubce.com"
     ]
   }
   ```
2. **重启 Docker 服务**  
   执行以下命令使配置生效：
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```
3. **重新拉取镜像**  
   再次尝试拉取 RocketMQ 镜像：
   ```bash
   sudo docker pull apache/rocketmq:5.3.3
   ```
---
### 三、使用代理（可选）
如果你使用了代理，确保 Docker 也通过代理访问网络：
1. **创建代理配置目录**  
   ```bash
   sudo mkdir -p /etc/systemd/system/docker.service.d
   ```
2. **添加代理配置文件**  
   创建 `/etc/systemd/system/docker.service.d/http-proxy.conf`，内容如下：
   ```ini
   [Service]
   Environment="HTTP_PROXY=http://proxy-address:port"
   Environment="HTTPS_PROXY=http://proxy-address:port"
   ```
3. **重启 Docker 服务**  
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```
---

