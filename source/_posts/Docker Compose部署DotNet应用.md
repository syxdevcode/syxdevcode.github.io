---
title: Docker Compose部署DotNet应用
date: 2022-09-07 10:38:06
tags:
  - Sql Server
  - DotNetCore
  - Docker
  - Docker Compose
  - 配置中心
categories:
  - DotNetCore
---

## 创建镜像

进入应用目录(存在 Dockerfile 文件)：

```Dockerfile
docker build -t lims:0.0.2 .
```

测试

```shell
# –rm :退出时，删除容器
docker run --name lims-server -e TZ=Asia/Shanghai -p 80:80 -d lims:0.0.2

# 多行命令
docker run \
--name lims-server \
-e TZ=Asia/Shanghai \
-p 80:80 \
-d lims:0.0.2
```

<!--more-->

## Docker Compose 编排

目录结构：

```shell
.
├── docker-compose.yml
└── var.env
```

`docker-compose.yml`文件：

```yml
version: "3"
services:
  lims_node1:
    image: lims:0.0.2
    restart: always
    container_name: lims_node1
    ports:
      - "5000:80"
    networks:
      - net0
    volumes:
      - /etc/localtime:/etc/localtime
    environment:
      - adminConsole=true
    env_file:
      - ./var.env
  lims_node2:
    image: lims:0.0.2
    restart: always
    container_name: lims_node2
    ports:
      - "5001:80"
    networks:
      - net0
    volumes:
      - /etc/localtime:/etc/localtime
    env_file:
      - ./var.env
  lims_node3:
    image: lims:0.0.2
    restart: always
    container_name: lims_node3
    ports:
      - "5002:80"
    networks:
      - net0
    volumes:
      - /etc/localtime:/etc/localtime
    env_file:
      - ./var.env
networks:
  net0:
```

`var.env` 文件：

```environment
TZ=Asia/Shanghai
AgileConfig__secret=ulxnjRq5jSSs
AgileConfig__env=TEST
```

## 命令

```shell
# 启动
docker compose up -d

# 移除
docker-compose down -v

# 重启
docker-compose restart


```
