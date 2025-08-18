# docker-compose 说明

## 前言
 在我们日常开发过程中，有时候需要临时整一个开发或者测试环境在同一台机器上面，这时候docker-compose 编排就是一个很好的选择，下面就来看看docker-compose的使用方法

 示例

```
version: "3.8"
networks:
  wms-common-network:
    driver: bridge

services:
  wms-common-db:
    image: mysql:8.0.37
    container_name: wms-common-db
    restart: always
    networks:
      - wms-common-network
    env_file:
      - .env
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - ./data/mysql-data:/var/lib/mysql
      - ./deployments/mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3307:3306"
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  wms-common-redis:
    image: redis:7.2-alpine
    container_name: wms-common-redis
    restart: always
    networks:
      - wms-common-network
    ports:
      - '16379:6379'
    command: redis-server --requirepass NJ_redis_ho*7@706$ --maxmemory 1gb --maxmemory-policy allkeys-lru
    volumes:
      - './data/redis-data:/data'
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  wms-common-admin-backend:
    image: wms_common_sysadm_api:latest
    container_name: wms-common-admin-backend
    restart: always
    networks:
      - wms-common-network
    env_file:
      - .env
    volumes:
      - ./deployments/backend/admin/config.yaml:/app/config.yaml
      - ./logs/admin/log:/app/log
    depends_on:
      wms-common-db:
        condition: service_healthy
      wms-common-redis:
        condition: service_healthy
      wms-common-rmqbroker:
        condition: service_healthy
    expose:
      - "3301"
    ports:
      - "3301:3301"

  wms-common-admin-nginx:
    image: nginx:1.29.0-alpine
    container_name: wms-common-admin-nginx
    restart: always
    networks:
      - wms-common-network
    env_file:
      - .env
    volumes:
      - ./deployments/nginx/admin/default.conf:/etc/nginx/conf.d/default.conf
      - ./deployments/frontend/admin/dist:/usr/share/nginx/html
      - ./logs/nginx/admin:/var/log/nginx  
    depends_on:
      - wms-common-admin-backend
    ports:
      - "8980:80"
      # - "3303:3301"

  wms-common-user-backend:
    image: wms_common_member_api:latest
    container_name: wms-common-user-backend
    restart: always
    networks:
      - wms-common-network
    env_file:
      - .env
    volumes:
      - ./deployments/backend/user/config.yaml:/app/config.yaml
      - ./logs/user/log:/app/log
    depends_on:
      wms-common-db:
        condition: service_healthy
      wms-common-redis:
        condition: service_healthy
    expose:
      - "3302"
    ports:
      - "3302:3302"

  wms-common-user-nginx:
    image: nginx:1.29.0-alpine
    container_name: wms-common-user-nginx
    restart: always
    networks:
      - wms-common-network
    env_file:
      - .env
    volumes:
      - ./deployments/nginx/user/default.conf:/etc/nginx/conf.d/default.conf
      - ./deployments/frontend/user/dist:/usr/share/nginx/html
      - ./logs/nginx/user:/var/log/nginx
    depends_on:
      - wms-common-user-backend
    ports:
      - "8981:80"
      # - "3302:3302"

     # RocketMQ服务配置（普通模式版）
  wms-common-rmqnamesrv:
    image: apache/rocketmq:5.3.2
    container_name: wms-common-rmqnamesrv
    ports:
      - 9876:9876
    networks:
      - wms-common-network
    command: sh mqnamesrv
    volumes:  
      - ./logs/rocketmq-namesrv/logs:/home/rocketmq/logs
    mem_limit: 512m  # 内存限制
    cpus: '0.5'   # CPU限制
    # --- 新增健康检查 ---
    healthcheck:
      # 检查命令：使用 Bash 尝试连接到 9876 端口
      # 如果连接成功，echo $? 会得到 0
      # 如果连接失败，echo $? 会得到非 0
      # test: ["CMD", "bash", "-c", "timeout 2 bash -c '</dev/tcp/localhost/9876' && echo 0 || echo 1"]
      # 更简洁的写法：
      test: ["CMD-SHELL", "timeout 2 bash -c '</dev/tcp/localhost/9876'"]
      # 检查间隔：5秒
      interval: 5s
      # 超时时间：3秒
      timeout: 3s
      # 连续失败 3 次后，才判定为不健康
      retries: 3
      # 启动后等待 10 秒再开始第一次检查
      start_period: 10s

  wms-common-rmqbroker:
    image: apache/rocketmq:5.3.2
    container_name: wms-common-rmqbroker
    ports:
      - 10909:10909
      - 10911:10911
      - 10912:10912
    environment:
      - NAMESRV_ADDR=wms-common-rmqnamesrv:9876
      # -e "NAMESRV_ADDR=wms-common-rmqnamesrv:9876" # 这行很重要，让 Broker 知道 NameServer 地址
      - "JAVA_OPT_EXT=-server -Xms256m -Xmx256m -Xmn128m"
    networks:
      - wms-common-network
    command: sh mqbroker -c /home/rocketmq/rocketmq-5.3.2/conf/broker.conf
    user: root
    volumes: 
      - ./deployments/rocketmq/broker.conf:/home/rocketmq/rocketmq-5.3.2/conf/broker.conf
      - ./data/rocketmq/store:/home/rocketmq/store
      - ./logs/rocketmq-broker/logs:/home/rocketmq/logs
    mem_limit: 2g     # Broker需要更多内存
    cpus: '1'      # 分配1个CPU核心
    depends_on:
    # --- 新增健康检查依赖 ---
      # 等待 rmqnamesrv 服务进入 healthy 状态后再启动
      wms-common-rmqnamesrv:
        condition: service_healthy
    healthcheck:
      # 核心检查命令：通过 mqadmin 查询自身状态
      test: ["CMD", "sh", "-c", "/home/rocketmq/rocketmq-5.3.2/bin/mqadmin brokerStatus -n rmqnamesrv:9876 -b 127.0.0.1:10911"]
      interval: 10s
      timeout: 10s
      retries: 3
      start_period: 30s # Broker 启动和注册可能需要更长时间，所以给足时间

  # wms-common-rmqproxy:  # 代理模式,这个代理服务用不到，一般是grpc客户端才用到，我们用的v2版本可以直接通过nameserver数组进行连接
  #   image: apache/rocketmq:5.3.2
  #   container_name: wms-common-rmqproxy
  #   networks:
  #     - wms-common-network
  #   ports:
  #     - 8080:8080
  #     - 8081:8081
  #   environment:
  #     - NAMESRV_ADDR=rmqnamesrv:9876
  #   command: sh mqproxy
  #   mem_limit: 1g     # Proxy中等内存需求
  #   cpus: '0.5'
  #   depends_on:
  #     - wms-common-rmqbroker
  #     - wms-common-rmqnamesrv
```
