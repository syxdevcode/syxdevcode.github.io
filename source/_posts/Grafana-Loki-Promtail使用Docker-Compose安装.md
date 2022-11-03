---
title: Grafana-Loki-Promtail使用Docker-Compose安装
date: 2022-10-24 17:50:52
tags:
  - Ubuntu
  - Grafana
  - Loki
  - Promtail
  - Docker Compose
  - Docker
categories:
  - Grafana
---

## 创建对应目录

```sh
mkdir -p /opt/grafana/loki/config
mkdir -p /opt/grafana/loki/data
mkdir -p /opt/grafana/promtail/log
mkdir -p /opt/grafana/conf
mkdir -p /opt/grafana/data
mkdir -p /opt/grafana/log
```

## 创建配置文件

### loki配置文件

路径：`/opt/grafana/loki/config/local-config.yaml` :

参考：[Configuring Grafana Loki](https://grafana.com/docs/loki/latest/configuration/)

<!--more-->
```yml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_server_max_recv_msg_size: 157286400  # 150M 可接收的最大 gRPC 消息大小
  grpc_server_max_send_msg_size: 157286400  # 150M 可以发送的最大 gRPC 消息大小

common:
  path_prefix: /loki
  storage:
    filesystem:
      chunks_directory: /loki/chunks
      rules_directory: /loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: loki_index_
        period: 24h

limits_config:
  retention_period: 5760h
  ingestion_rate_strategy: local  
  ingestion_rate_mb: 15 # 每个用户每秒的采样率限制
  ingestion_burst_size_mb: 30 # 每个用户允许的采样突发大小
  reject_old_samples: true   # 是否拒绝旧样本
  reject_old_samples_max_age: 4440h   # 4440h小时之前的样本被拒绝
  max_query_length: 4440h  # 块存储查询的长度限制。禁用=0h，默认default = 721h

storage_config:
  boltdb_shipper:
    active_index_directory: /loki/boltdb-shipper-active
    cache_location: /loki/boltdb-shipper-cache
    cache_ttl: 24h
    shared_store: filesystem
  filesystem:
    directory: /loki/chunks

compactor:
  working_directory: /loki/retention
  shared_store: filesystem
  compaction_interval: 60m
  retention_enabled: true
  retention_delete_delay: 72h
  retention_delete_worker_count: 150
  
table_manager:
  retention_deletes_enabled: true   # 保留删除开启
  retention_period: 5760h  # 超过5760h的块数据将被删除

chunk_store_config:
  max_look_back_period: 4440h # 限制可以查询回溯数据的长度

ruler:
  alertmanager_url: http://localhost:9093
```

### promtail配置文件

路径：

参考：[https://grafana.com/docs/loki/latest/clients/promtail/configuration/](https://grafana.com/docs/loki/latest/clients/promtail/configuration/)

```yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0
  grpc_server_max_recv_msg_size: 157286400 # 150M
  grpc_server_max_send_msg_size: 157286400 # 150M

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
- job_name: system
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log
```

## 创建用户和用户组

参考：

[Run Grafana Docker image](https://grafana.com/docs/grafana/latest/setup-grafana/installation/docker/)

[docker环境下的Grafana安装](https://www.cnblogs.com/sfccl/p/12936282.html)

1，宿主机新增用户 `grafana`，并修改UID和GID都为472（因为容器内的运行用户也是grafana，且UID和GID都是472）

```sh
useradd grafana
vi /etc/passwd # 编辑用户Id

# 参考： -------
grafana:x:472:472::/home/grafana:/bin/sh
```

2，编辑用户组

```sh
vi /etc/group

# 参考： -------
grafana:x:472:
```

3，修改目录所有者为`grafana`

```sh
chown -R grafana:grafana /opt/grafana/conf && \
chmod -R a+r /opt/grafana/conf && \
chmod -R a+r /opt/grafana/data && \
chown -R grafana:grafana /opt/grafana/data
```

Loki数据目录权限：

```sh
chmod 777 /opt/grafana/loki/data
```

## 创建docker网络

```sh
docker network create --driver bridge --subnet 10.10.13.0/24 --gateway 10.10.13.1 docker_compose_net
```

## 编排docker-compose

```yml
version: "3"

networks:
  default:
    external:
      name: docker_compose_net

services:
  loki:
    image: grafana/loki:2.6.0
    restart: unless-stopped
    container_name: loki
    ports:
      - "3100:3100"
    volumes:
      - /opt/grafana/loki/:/etc/loki/
      - /opt/grafana/loki/data:/loki/
      - /opt/grafana/loki/config/:/etc/loki/config
    command: -config.file=/etc/loki/config/local-config.yaml
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海

  promtail:
    image: grafana/promtail:2.6.0
    restart: unless-stopped
    container_name: promtail
    volumes:
      - /opt/grafana/promtail/log:/var/log
      - /opt/grafana/promtail/:/etc/promtail/
    command: -config.file=/etc/promtail/config.yml
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海

  grafana:
    image: grafana/grafana:latest
    restart: unless-stopped
    container_name: grafana
    volumes:
      # - /opt/grafana/conf/:/etc/grafana/ # 不指定配置文件,使用默认配置文件
      - /opt/grafana/data/:/var/lib/grafana/
      - /opt/grafana/log/:/var/log/grafana/
    ports:
      - "3000:3000"
    user: '472'
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海
```

## 启动&移除

```sh
# 启动
docker-compose up -d

# 移除
docker-compose down -v
```
