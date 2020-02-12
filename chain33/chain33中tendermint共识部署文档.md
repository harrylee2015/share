# chain33中tendermint共识部署文档

## 基于政务平台生成的安装包的分布式部署

1.通过政务平台，创建任务，添加节点，生成安装包
  略
2.各个节点下载上一步生成的专属安装包

3.解压安装包

 ```
 tar -xvf xxx.tar.gz
 ```
 
 
4.启动服务
 
 ```
  #chain33和监控agent一起启动 
  bash start.sh all
  
  #只启动chain33
  bash start.sh chain33
  
 ```
 
 4.停止服务
 
 ```
  #停掉所有
  bash stop.sh all
 ```

## 基于docker-compose 方式单机版伪分布式部署

*目前只支持在ubuntu系统上面部署*

```
 1.下载安装包
 
 weget https://zqpt-test.oss-cn-shanghai.aliyuncs.com/tendermint/chain33_tendermint_v6.3.0_docker-compose.tar.gz
 
 
 2.解压安装包
 
 tar -xvf chain33_tendermint_v6.3.0_docker-compose.tar.gz
 
 3.进入docker目录执行安装脚本（为了避免权限问题，以下命令请用root用户去执行）
 
 cd  docker
 
 4.安装docker及docker-compose
 
  bash docker-install.sh
  
 5.本地构建服务镜像
 
  bash docker-build.sh
  
 6.启动服务
 
 docker-compose up 

 或者后台启动
 
 docker-compose up -d
 
 
 7.停掉服务
 
  docker-compose down
 
```
### 说明

docker-compose方式部署网络端口映射

节点名称|集群内网ip|jrpc服务端口射到本地物理机服务端口号
--|--|--
chain33-tendermint0|172.30.0.2|8801
chain33-tendermint1|172.30.0.3|8701
chain33-tendermint2|172.30.0.4|8601
chain33-tendermint3|172.30.0.5|8501

访问容器节点，可通过jrpc映射的物理机端口8801，8701，8601，8501进行访问

 
