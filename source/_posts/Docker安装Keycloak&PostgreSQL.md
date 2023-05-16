---
title: Docker安装Keycloak&PostgreSQL
date: 2023-05-11 17:21:31
tags:
  - Docker Compose
  - Docker
  - Ubuntu
  - Linux
  - Keycloak
  - PostgreSQL
  - 安装部署
categories: 
  - Keycloak
---

## 构建Keycloak镜像

编写 `Dockerfile` 文件：

```sh
FROM quay.io/keycloak/keycloak:latest as builder

WORKDIR /opt/keycloak
# for demonstration purposes only, please make sure to use proper certificates in production instead
RUN keytool -genkeypair -storepass password -storetype PKCS12 -keyalg RSA -keysize 2048 -dname "CN=server" -alias server -ext "SAN:c=DNS:localhost,IP:127.0.0.1" -keystore conf/server.keystore
RUN /opt/keycloak/bin/kc.sh build

FROM quay.io/keycloak/keycloak:latest
COPY --from=builder /opt/keycloak/ /opt/keycloak/

ENTRYPOINT ["/opt/keycloak/bin/kc.sh"]
```

```yml
docker build . -t mykeycloak:1.0.0
```

<!--more-->
## 配置

[postgres docker hub](https://hub.docker.com/_/postgres/tags)

### 创建外部网络

如果没有外部网络，需要运行以下命令创建：

```sh
docker network create --driver bridge --subnet 10.10.13.0/24 --gateway 10.10.13.1 docker_compose_net
```

### 创建postgres目录

```sh
mkdir pgdata
chmod 777 pgdata
```

目录结构：

```sh
tree -Lh 1
[4.0K]  .
├── [ 528]  Dockerfile
├── [1.1K]  compose.yml
└── [4.0K]  pgdata
```

### 配置文件

```yml
version: '3.9'

networks:
  default:
    external:
      name: docker_compose_net

services:
  postgres:
    image: "postgres:${POSTGRESQL_VERSION:?err}"
    restart: unless-stopped
    container_name: postgressql
    environment:
      POSTGRES_DB: ${POSTGRESQL_DB}
      POSTGRES_USER: ${POSTGRESQL_USER}
      POSTGRES_PASSWORD: ${POSTGRESQL_PASS}
    volumes:
      - /opt/keycloak/pgdata:/var/lib/postgresql/data
    ports:
      - ${POSTGRESQL_PORT}:5432

  keycloak:
    image: "${KC_DOCKER_IMAGE:?err}:${KC_VERSION:?err}"
    restart: unless-stopped
    command: ["start"]
    depends_on:
      - postgres
    container_name: keycloak
    environment:
      KC_DB: ${KC_DB}
      KC_DB_USERNAME: ${POSTGRESQL_USER}
      KC_DB_PASSWORD: ${POSTGRESQL_PASS}
      KC_DB_URL: "jdbc:postgresql://postgres:5432/keycloak"
      KC_METRICS_ENABLED: true
      KC_HEALTH_ENABLED: true
      KC_HOSTNAME: ${KC_HOSTNAME}
      KC_HOSTNAME_PORT: ${KC_HOSTNAME_PORT}
      KEYCLOAK_ADMIN: ${KEYCLOAK_ADMIN}
      KEYCLOAK_ADMIN_PASSWORD: ${KEYCLOAK_ADMIN_PASSWORD}
    ports:
      - ${KC_HOSTNAME_PORT}:8443
```

`.env` 文件：

```sh
# https://www.postgresql.org/docs/release
POSTGRESQL_VERSION=15.3-alpine3.18
POSTGRESQL_DB=keycloak
POSTGRESQL_USER=KcSuAdmin
POSTGRESQL_PASS=keycloak
POSTGRESQL_PORT=5432

# https://github.com/keycloak/keycloak/releases
KC_DOCKER_IMAGE=mykeycloak
KC_VERSION=1.0.0
KC_DB=postgres
KC_HOSTNAME=10.10.0.106
KC_HOSTNAME_PORT=9800
KEYCLOAK_ADMIN=admin
KEYCLOAK_ADMIN_PASSWORD=keycloak
```

## 启动停止

```yml
# 启动
docker-compose -f /opt/keycloak/compose.yml up -d

# 移除
docker-compose -f /opt/keycloak/compose.yml down -v

# 更新docker-compose
docker-compose -f /opt/keycloak/compose.yml up -d --build
```

参考：

[Running Keycloak in a container](https://www.keycloak.org/server/containers#_running_a_standard_keycloak_container)

[Keycloak All configuration](https://www.keycloak.org/server/all-config)

[https://github.com/eabykov/keycloak-compose/blob/main/compose.yml](https://github.com/eabykov/keycloak-compose/blob/main/compose.yml)

[Keycloak With Docker Compose](https://gruchalski.com/posts/2020-09-03-keycloak-with-docker-compose)
