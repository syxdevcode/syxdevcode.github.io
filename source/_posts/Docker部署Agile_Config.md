---
title: Docker部署Agile_Config
date: 2022-09-06 14:37:53
tags:
  - Sql Server
  - DotNetCore
  - Docker
  - Docker Compose
  - 命名实例
  - 配置中心
categories:
  - DotNetCore
---

## Docer Compose 配置

### 创建外部网络

如果没有外部网络，需要运行以下命令创建：

```sh
# 查看网络列表
docker network ls

# 创建网络
docker network create --driver bridge --subnet 10.10.13.0/24 --gateway 10.10.13.1 docker_compose_net
```

```yml
version: "3"

networks:
  default:
    external:
      name: docker_compose_net

services:
  agile_config_admin:
    image: "kklldog/agile_config:v-1.6.14"
    container_name: agile-admin
    restart: unless-stopped
    ports:
      - "15000:5000"
    volumes:
      - /etc/localtime:/etc/localtime
    environment:
      - adminConsole=true
    env_file:
      - ./var.env
  agile_config_node1:
    image: "kklldog/agile_config:v-1.6.14"
    container_name: agile-node1
    restart: unless-stopped
    ports:
      - "15001:5000"
    volumes:
      - /etc/localtime:/etc/localtime
    env_file:
      - ./var.env
    depends_on:
      - agile_config_admin
  agile_config_node2:
    image: "kklldog/agile_config:v-1.6.14"
    container_name: agile-node2
    restart: unless-stopped
    ports:
      - "15002:5000"
    volumes:
      - /etc/localtime:/etc/localtime
    env_file:
      - ./var.env
    depends_on:
      - agile_config_admin
```

<!--more-->

注意：

- 1，docker-compose 中 `$` 会进行插值操作，如果连接串中有 `$` 符号，需要引入外部环境变量文件。
- 2，注意数据库链接服务器地址: 单反斜杠(不推荐)，命名实例推荐端口号方式。

与 `docker-compose.yml` 同目录下：

`var.env` 内容

```env_file
TZ=Asia/Shanghai
cluster=true
db__provider=sqlserver
db__provider= Encrypt=True;TrustServerCertificate=True;Persist Security Info=False; User ID=ConfigServer; Password=RlDjqOpF$; Initial Catalog=ConfigCenter; Server=10.10.0.36,1435;
```

目录结构如下：

```shell
/lims/agile_config
├── docker-compose.yml
└── var.env
```

## Docker&Docker Compose 命令

```shell
# 启动
docker-compose up -d

# 移除
docker-compose down -v

# 查看容器信息
docker inspect agile_config-agile_config_admin-1

# 重启
docker-compose restart

# 查看容器内文件
docker exec lims-server tail -n +0 AdminNETConfig.json
```
