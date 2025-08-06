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
    depends_on:
      - wms-common-user-backend
    ports:
      - "8981:80"
      # - "3302:3302"

```
