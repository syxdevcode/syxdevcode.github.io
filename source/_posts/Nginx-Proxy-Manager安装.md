---
title: Nginx-Proxy-Manager安装
date: 2023-05-23 18:13:19
tags:
  - Docker Compose
  - Docker
  - Nginx
  - Ubuntu
  - Linux
  - Nginx-Proxy-Manager
categories:
  - Nginx-Proxy-Manager
---

## 配置

### 创建外部网络

如果没有外部网络，需要运行以下命令创建：

```sh
# 查看网络列表
docker network ls

# 创建网络
docker network create --driver bridge --subnet 10.10.13.0/24 --gateway 10.10.13.1 docker_compose_net
```

### 配置文件

创建目录：

```sh
mkdir -p nginx-proxy-manager/data nginx-proxy-manager/letsencrypt
```

```yml
version: '3.8'

networks:
  default:
    external:
      name: docker_compose_net

services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    container_name: nginx-proxy-manager
    ports:
      - '9850:80'
      - '9851:81'
      - '9843:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

```sh
# 启动
docker-compose -f /opt/nginx-proxy-manager/compose.yml up -d

# 重启
docker-compose -f /opt/nginx-proxy-manager/compose.yml restart

# 移除
docker-compose -f /opt/nginx-proxy-manager/compose.yml down -v

# 更新
docker-compose -f /opt/nginx-proxy-manager/compose.yml up -d --build
```

容器81端口为管理端口。

Default Admin User:

Email:    admin@example.com
Password: changeme

参考：

[https://github.com/NginxProxyManager/nginx-proxy-manager](https://github.com/NginxProxyManager/nginx-proxy-manager)
