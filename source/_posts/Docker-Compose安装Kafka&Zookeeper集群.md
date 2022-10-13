---
title: Docker-Compose安装Kafka&Zookeeper集群
date: 2022-09-26 17:42:57
tags:
  - Ubuntu
  - Kafka
  - 消息队列
  - Zookeeper
  - Linux
  - 安装部署
  - 分布式
  - Docker Compose
  - Docker
categories:
  - Kafka
---

[kafka镜像](https://hub.docker.com/r/wurstmeister/kafka/tags)

[zookeeper镜像](https://hub.docker.com/_/zookeeper/tags)

## 创建docker网络

```sh
# 命令创建Docker网络
docker network create zookeeper_network
# 查看 Docker网络
docker network ls
```

<!--more-->
## Zookeeper集群

目录结构：

```sh
├── docker-compose.yml
├── zoo1
│   ├── data
│   └── datalog
├── zoo2
│   ├── data
│   └── datalog
└── zoo3
    ├── data
    └── datalog
```

编辑 docker-compose.yml 配置文件：

`vim docker-compose.yml`

```yml
version: '3.8'

networks:
  default:
    external:
      name: zookeeper_network
services:
  zoo1:
    image: zookeeper:latest
    container_name: zoo1
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    volumes:
      - "/opt/zookeeper/zoo1/data:/data"
      - "/opt/zookeeper/zoo1/datalog:/datalog"
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
  zoo2:
    image: zookeeper:latest
    container_name: zoo2
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    volumes:
      - "/opt/zookeeper/zoo2/data:/data"
      - "/opt/zookeeper/zoo2/datalog:/datalog"
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181
  zoo3:
    image: zookeeper:latest
    container_name: zoo3
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    volumes:
      - "/opt/zookeeper/zoo3/data:/data"
      - "/opt/zookeeper/zoo3/datalog:/datalog"
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
```

## Kafka集群

目录结构如下：

```sh
.
├── data1
├── data2
├── data3
└── docker-compose.yml
```

编辑 docker-compose.yml 配置文件：

`vim docker-compose.yml`

```yml
version: '3.8'

networks:
  default:
    external:
      name: zookeeper_network

services:
  kafka1:
    image: wurstmeister/kafka:2.13-2.8.1
    restart: unless-stopped
    container_name: kafka1
    hostname: kafka1
    ports:
      - "9092:9092"
    external_links:
      - zoo1
      - zoo2
      - zoo3
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://10.10.0.106:9092    ## 宿主机IP
      KAFKA_ADVERTISED_HOST_NAME: kafka1
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2181,zoo3:2181"
      KAFKA_LOG_RETENTION_HOURS: 120
      KAFKA_MESSAGE_MAX_BYTES: 10000000
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10000000
      KAFKA_GROUP_MAX_SESSION_TIMEOUT_MS: 60000
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DELETE_RETENTION_MS: 1000
    volumes:
      - "/opt/kafka/data1/:/kafka"
  kafka2:
    image: wurstmeister/kafka:2.13-2.8.1
    restart: unless-stopped
    container_name: kafka2
    hostname: kafka2
    ports:
      - "9093:9092"
    external_links:
      - zoo1
      - zoo2
      - zoo3
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://10.10.0.106:9093    ## 宿主机IP
      KAFKA_ADVERTISED_HOST_NAME: kafka2
      KAFKA_ADVERTISED_PORT: 9093
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2181,zoo3:2181"
      KAFKA_LOG_RETENTION_HOURS: 120
      KAFKA_MESSAGE_MAX_BYTES: 10000000
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10000000
      KAFKA_GROUP_MAX_SESSION_TIMEOUT_MS: 60000
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DELETE_RETENTION_MS: 1000
    volumes:
      - "/opt/kafka/data2/:/kafka"
  kafka3:
    image: wurstmeister/kafka:2.13-2.8.1
    restart: unless-stopped
    container_name: kafka3
    hostname: kafka3
    ports:
      - "9094:9092"
    external_links:
      - zoo1
      - zoo2
      - zoo3
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://10.10.0.106:9094   ## 宿主机IP
      KAFKA_ADVERTISED_HOST_NAME: kafka3
      KAFKA_ADVERTISED_PORT: 9094
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2181,zoo3:2181"
      KAFKA_LOG_RETENTION_HOURS: 120
      KAFKA_MESSAGE_MAX_BYTES: 10000000
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10000000
      KAFKA_GROUP_MAX_SESSION_TIMEOUT_MS: 60000
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DELETE_RETENTION_MS: 1000
    volumes:
      - "/opt/kafka/data3/:/kafka"
  kafka-manager: # Kafka 图形管理界面
    image: sheepkiller/kafka-manager:latest
    restart: unless-stopped
    container_name: kafka-manager
    hostname: kafka-manager
    ports:
      - "9000:9000"
    links:            # 连接本compose文件创建的container
      - kafka1
      - kafka2
      - kafka3
    external_links:   # 连接外部compose文件创建的container
      - zoo1
      - zoo2
      - zoo3
    environment:
      ZK_HOSTS: zoo1:2181,zoo2:2181,zoo3:2181
      KAFKA_BROKERS: kafka1:9092,kafka2:9093,kafka3:9094
```

## 测试运行状态

```sh
# 查看容器运行状态
docker ps

# 测试生成者
docker exec -it kafka1 /bin/bash
kafka-console-producer.sh --broker-list kafka1:9092,kafka2:9092,kafka3:9092 --topic test

# 测试消费者
docker exec -it kafka2 /bin/bash
kafka-console-consumer.sh --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092 --topic test --from-beginning
```

参考：

[docker-compose安装kafka集群并解决docker内kafka外界无法访问问题](https://blog.csdn.net/qq_39526294/article/details/124293954)

[Docker搭建带SASL用户密码验证的Kafka](https://www.cnblogs.com/zhouj850/p/15630101.html)

[docker-compose配置带密码验证的kafka](https://www.cnblogs.com/zhouj850/p/15683605.html)