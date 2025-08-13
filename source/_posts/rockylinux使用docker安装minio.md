---
title: rockylinux使用docker安装minio
date: 2022-09-19 10:23:27
tags:
  - Docker Compose
  - Docker
  - RockyLinux
  - MinIo
  - Linux
categories:
  - MinIo
---

## 单节点编排

```yml
version: "3"
services:
  minio:
    restart: always
    image: minio/minio
    container_name: minio
    ports:
      - "9100:9000" # api 端口
      - "9190:9090" # 控制台端口
    networks:
      - net0
    volumes:
      - /opt/minio/data:/data
    privileged: true
    environment:      
      MINIO_ACCESS_KEY: admin    #管理后台用户名
      MINIO_SECRET_KEY: admin123 #管理后台密码，最小8个字符
    command: server --console-address ':9090' /data  #指定容器中的目录 /data
networks:
  net0:
```

执行：

```sh
# 启动
docker-compose up -d

# 移除
docker-compose down -v
```

<!--more-->
## 群集编排

```yml
version: '3'

# starts 4 docker containers running minio server instances.
# using nginx reverse proxy, load balancing, you can access
# it through port 9000.
services:
  minio1:
    image: minio/minio
    hostname: minio1
    volumes:
      - data1-1:/data1
      - data1-2:/data2
    expose:
      - "9000"
      - "9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio123
    command: server --console-address ":9001" http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio2:
    image: minio/minio
    hostname: minio2
    volumes:
      - data2-1:/data1
      - data2-2:/data2
    expose:
      - "9000"
      - "9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio123
    command: server --console-address ":9001" http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio3:
    image: minio/minio
    hostname: minio3
    volumes:
      - data3-1:/data1
      - data3-2:/data2
    expose:
      - "9000"
      - "9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio123
    command: server --console-address ":9001" http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  minio4:
    image: minio/minio
    hostname: minio4
    volumes:
      - data4-1:/data1
      - data4-2:/data2
    expose:
      - "9000"
      - "9001"
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minio123
    command: server --console-address ":9001" http://minio{1...4}/data{1...2}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3

  nginx:
    image: nginx:1.19.2-alpine
    hostname: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "9000:9000"
      - "9001:9001"
    depends_on:
      - minio1
      - minio2
      - minio3
      - minio4

## By default this config uses default local driver,
## For custom volumes replace with volume driver configuration.
volumes:
  data1-1:
  data1-2:
  data2-1:
  data2-2:
  data3-1:
  data3-2:
  data4-1:
  data4-2:
```

参考：

[docker-compose 搭建 minio 分布式对象存储](https://blog.csdn.net/weixin_40461281/article/details/118568289)