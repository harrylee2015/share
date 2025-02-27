# docker-compose 说明

## 前言
 在我们日常开发过程中，有时候需要临时整一个开发或者测试环境在同一台机器上面，这时候docker-compose 编排就是一个很好的选择，下面就来看看docker-compose的使用方法

 示例

```
version: "3"
networks:
  license-network:
    driver: bridge

services:
  license-server-backend-tls:
    image: git.tongyuan.cc:5050/license/mworks-license/license-service:v2.0.0
    container_name: mworks-license-backend-tls
    restart: always
    networks:
      - license-network
    env_file:
      - .tls_env
    volumes:
      - ./tls-logs:/logs
    logging:
      driver: json-file
      options:
        max-size: "200k"
        max-file: "10"
    expose:
      - "3301"
    ports:
      - "4303:3301"

  license-server-backend:
    image: git.tongyuan.cc:5050/license/mworks-license/license-service:v2.0.0
    container_name: mworks-license-backend
    restart: always
    networks:
      - license-network
    env_file:
      - .env
    volumes:
      - ./logs:/logs
    logging:
      driver: json-file
      options:
        max-size: "200k"
        max-file: "10"
    expose:
      - "3301"
    # ports:
    #   - "3301:3301"

  license-server-nginx:
    image: git.tongyuan.cc:5050/license/mw-license-deploy:main
    container_name: mworks-license-nginx
    restart: always
    networks:
      - license-network
    env_file:
      - .env
    volumes:
      - ./dist:/usr/share/nginx/html
    logging:
      driver: json-file
      options:
        max-size: "200k"
        max-file: "10"
    expose:
      - "80"
      - "3301"
    ports:
      - "8080:80"
      - "3303:3301"

  license-server-db:
    image: mysql:8.0.37
    container_name: mworks-license-db
    restart: always
    networks:
      - license-network
    env_file:
      - .env
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - ./db_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "3307:3306"

  license-server-redis:
    image: 'bitnami/redis:7.2'
    networks:
      - license-network
    ports:
      - '6379:6379'
    environment:
      - REDIS_PASSWORD=TR123!
      - REDIS_DISABLE_COMMANDS=FLUSHDB,FLUSHALL
      - REDIS_MAXMEMORY=1g  # 设置最大内存为1GB
        # volumes:
            # - './redis-data:/bitnami/redis/data'
        #   command: /run.sh --appendonly yes

```