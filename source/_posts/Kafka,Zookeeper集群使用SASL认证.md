---
title: Kafka,Zookeeper集群使用SASL认证
date: 2022-09-27 13:02:45
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

## SASL认证简介

SASL：简单认证和安全层

SASL是一种用来扩充C/S模式验证能力的机制认证机制，全称 `Simple Authentication and Security Layer`。

当你设定sasl时，你必须决定两件事；一是用于交换“标识信 息”（或称身份证书）的验证机制；一是决定标识信息存储方法的验证架构。

sasl验证机制规范client与server之间的应答过程以及传输内容的编码方法，sasl验证架构决定服务器本身如何存储客户端的身份证书以及如何核验客户端提供的密码。

如果客户端能成功通过验证，服务器端就能确定用户的身份， 并借此决定用户具有怎样的权限。

比较常用的机制plain：plain是最简单的机制，但同时也是最危险的机制，因为身份证书（登录名称与密码）是以base64字符串格式通过网络，没有任何加密保护措施。因此，使用plain机制时，可能会想要结合tls。

参考：

[SASL - 简单认证和安全层](https://www.cnblogs.com/yanwei-wang/p/4679950.html)

<!--more-->
## 新建Sasl配置文件

/opt/sasl 目录下新建 `server_jaas.conf`:

`vim server_jaas.conf`：

```cnf
Client {
    org.apache.zookeeper.server.auth.DigestLoginModule required
    username="admin"
    password="12345678";
};

Server {
    org.apache.zookeeper.server.auth.DigestLoginModule required
    username="admin"
    password="12345678"
    user_super="12345678"
    user_admin="12345678";
};


KafkaServer {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="12345678"
    user_admin="12345678";
};

KafkaClient {
    org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="12345678";
};
```

通过`org.apache.kafka.common.security.plain.PlainLoginModule`由指定采用PLAIN机制，定义了用户。
创建多少个用户，可根据业务需要配置，用户名和密码可自定义设置。

* usemame和password指定该代理与集群其他代理初始化连接的用户名和密码;
* "user_"为前缀后接用户名方式创建连接代理的用户名和密码，例如，user_producer=“producerpwd” 是指用户名为producer，密码为producerpwd;
* username为admin的用户，和user为admin的用户，密码要保持一致，否则会认证失败。

## Zookeeper群集配置

目录结构：

```sh
.
├── -
├── conf
│   └── zoo.cfg
├── docker-compose.yml
├── zoo1
│   ├── data
│   └── datalog
├── zoo2
│   ├── data
│   └── datalog
└── zoo3
│   ├── data
│   └── datalog
```

注意：

ZooKeeper安装好之后，在安装目录的conf文件夹下可以找到一个名为 `zoo_sample.cfg` 的文件，是ZooKeeper配置文件的模板。

ZooKeeper启动时，会默认加载 `conf/zoo.cfg` 作为配置文件，所以需要将 `zoo_sample.cfg` 复制一份，命名为 `zoo.cfg` ，然后根据需要设定里面的配置项。

`docker-compose.yml` 配置如下：

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
      - "/opt/sasl:/apache-zookeeper-3.8.0-bin/conf"
    environment:
      TZ: Asia/Shanghai
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181
      SERVER_JVMFLAGS: -Djava.security.auth.login.config=/apache-zookeeper-3.8.0-bin/conf/server_jaas_zoo.conf
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
      - "/opt/sasl:/apache-zookeeper-3.8.0-bin/conf"
    environment:
      TZ: Asia/Shanghai
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181
      SERVER_JVMFLAGS: -Djava.security.auth.login.config=/apache-zookeeper-3.8.0-bin/conf/server_jaas_zoo.conf
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
      - "/opt/sasl:/apache-zookeeper-3.8.0-bin/conf"
    environment:
      TZ: Asia/Shanghai
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
      SERVER_JVMFLAGS: -Djava.security.auth.login.config=/apache-zookeeper-3.8.0-bin/conf/server_jaas_zoo.conf
```

`zoo.cfg` 配置如下：

`vim conf/zoo.cfg`

```cnf
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/opt/zookeeper-3.4.13/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
autopurge.purgeInterval=1

# zk3.6.0版本中添加，为true时要求客户端连接zk时必须进行SASL认证才可以连接成功，
# 也就是说没有进行SASL认证的匿名用户就无法连接了，相当于在连接时设置了一个登录密码
sessionRequireClientSASLAuth=true

authProvider.1=org.apache.zookeeper.server.auth.SASLAuthenticationProvider

requireClientAuthScheme=sasl

jaasLoginRenew=3600000

zookeeper.sasl.client=true
```

相关命令：

```sh
# 启动
docker-compose up -d

# 移除
docker-compose down -v
```

## Kafka集群配置

目录结构如下：

```sh
.
├── conf
│   └── application.conf
├── data1
├── data2
├── data3
└── docker-compose.yml
```

`application.conf` 配置如下：

```cnf
# Copyright 2015 Yahoo Inc. Licensed under the Apache License, Version 2.0
# See accompanying LICENSE file.

# This is the main configuration file for the application.
# ~~~~~

# Secret key
# ~~~~~
# The secret key is used to secure cryptographics functions.
# If you deploy your application to several instances be sure to use the same key!
play.crypto.secret="^<csmm5Fx4d=kwregwsr?k[mc;IZE<_asdfes_/7@afewqwe"
play.crypto.secret=${?APPLICATION_SECRET}

# The application languages
# ~~~~~
play.i18n.langs=["en"]

play.http.requestHandler = "play.http.DefaultHttpRequestHandler"
play.http.context = "/"
play.application.loader=loader.KafkaManagerLoader

kafka-manager.zkhosts="kafka-manager-zookeeper:2181"
kafka-manager.zkhosts=${?ZK_HOSTS}
pinned-dispatcher.type="PinnedDispatcher"
pinned-dispatcher.executor="thread-pool-executor"
application.features=["KMClusterManagerFeature","KMTopicManagerFeature","KMPreferredReplicaElectionFeature","KMReassignPartitionsFeature"]

akka {
  loggers = ["akka.event.slf4j.Slf4jLogger"]
  loglevel = "INFO"
}


basicAuthentication.enabled=true
basicAuthentication.enabled=${?KAFKA_MANAGER_AUTH_ENABLED}
basicAuthentication.username="admin"
basicAuthentication.username=${?KAFKA_MANAGER_USERNAME}
basicAuthentication.password="password"
basicAuthentication.password=${?KAFKA_MANAGER_PASSWORD}
basicAuthentication.realm="Kafka-Manager"


kafka-manager.consumer.properties.file=${?CONSUMER_PROPERTIES_FILE}
```

`docker-compose.yml` 配置如下：

`vim docker-compose.yml`:

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
      TZ: Asia/Shanghai
      KAFKA_BROKER_ID: 1
      #KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      #KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://10.10.0.106:9092    ## 宿主机IP
      KAFKA_LISTENERS: SASL_PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://10.10.0.106:9092
      KAFKA_ADVERTISED_HOST_NAME: kafka1
      KAFKA_ADVERTISED_PORT: 9092
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2181,zoo3:2181"
      KAFKA_LOG_RETENTION_HOURS: 120
      KAFKA_MESSAGE_MAX_BYTES: 10000000
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10000000
      KAFKA_GROUP_MAX_SESSION_TIMEOUT_MS: 60000
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DELETE_RETENTION_MS: 1000
      KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SASL_PLAINTEXT
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.auth.SimpleAclAuthorizer
      KAFKA_SUPER_USERS: User:admin
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true" #设置为true，ACL机制为黑名单机制，只有黑名单中的用户无法访问，默认为false，ACL机制为白名单机制，只有白名单中的用户可以访问
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_OPTS: -Djava.security.auth.login.config=/opt/kafka/conf/server_jaas.conf
    volumes:
      - "/opt/kafka/data1/:/kafka"
      - "/opt/sasl/:/opt/kafka/conf/"
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
      TZ: Asia/Shanghai
      KAFKA_BROKER_ID: 2
      #KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      #KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://10.10.0.106:9092    ## 宿主机IP
      KAFKA_LISTENERS: SASL_PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://10.10.0.106:9093
      KAFKA_ADVERTISED_HOST_NAME: kafka2
      KAFKA_ADVERTISED_PORT: 9093
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2181,zoo3:2181"
      KAFKA_LOG_RETENTION_HOURS: 120
      KAFKA_MESSAGE_MAX_BYTES: 10000000
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10000000
      KAFKA_GROUP_MAX_SESSION_TIMEOUT_MS: 60000
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DELETE_RETENTION_MS: 1000
      KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SASL_PLAINTEXT
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.auth.SimpleAclAuthorizer
      KAFKA_SUPER_USERS: User:admin
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true" #设置为true，ACL机制为黑名单机制，只有黑名单中的用户无法访问，默认为false，ACL机制为白名单机制，只有白名单中的用户可以访问
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_OPTS: -Djava.security.auth.login.config=/opt/kafka/conf/server_jaas.conf
    volumes:
      - "/opt/kafka/data2/:/kafka"
      - "/opt/sasl/:/opt/kafka/conf/"
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
      TZ: Asia/Shanghai
      KAFKA_BROKER_ID: 3
      #KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      #KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://10.10.0.106:9092    ## 宿主机IP
      KAFKA_LISTENERS: SASL_PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: SASL_PLAINTEXT://10.10.0.106:9094
      KAFKA_ADVERTISED_HOST_NAME: kafka3
      KAFKA_ADVERTISED_PORT: 9094
      KAFKA_ZOOKEEPER_CONNECT: "zoo1:2181,zoo2:2181,zoo3:2181"
      KAFKA_LOG_RETENTION_HOURS: 120
      KAFKA_MESSAGE_MAX_BYTES: 10000000
      KAFKA_REPLICA_FETCH_MAX_BYTES: 10000000
      KAFKA_GROUP_MAX_SESSION_TIMEOUT_MS: 60000
      KAFKA_NUM_PARTITIONS: 3
      KAFKA_DELETE_RETENTION_MS: 1000
      KAFKA_SECURITY_INTER_BROKER_PROTOCOL: SASL_PLAINTEXT
      KAFKA_SASL_MECHANISM_INTER_BROKER_PROTOCOL: PLAIN
      KAFKA_SASL_ENABLED_MECHANISMS: PLAIN
      KAFKA_AUTHORIZER_CLASS_NAME: kafka.security.auth.SimpleAclAuthorizer
      KAFKA_SUPER_USERS: User:admin
      KAFKA_ALLOW_EVERYONE_IF_NO_ACL_FOUND: "true" #设置为true，ACL机制为黑名单机制，只有黑名单中的用户无法访问，默认为false，ACL机制为白名单机制，只有白名单中的用户可以访问
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_OPTS: -Djava.security.auth.login.config=/opt/kafka/conf/server_jaas.conf
    volumes:
      - "/opt/kafka/data3/:/kafka"
      - "/opt/sasl/:/opt/kafka/conf/"
  kafka-ui: # Kafka 图形管理界面
    image: provectuslabs/kafka-ui
    restart: unless-stopped
    container_name: kafka-ui
    hostname: kafka-ui
    ports:
      - "9000:8080"
    links:            # 连接本compose文件创建的container
      - kafka1
      - kafka2
      - kafka3
    environment:
      SERVER_SERVLET_CONTEXT_PATH: /kafkaui # 访问地址：host:port/kafkaui/ui
      AUTH_TYPE: "LOGIN_FORM"
      SPRING_SECURITY_USER_NAME: suPerAdmin
      SPRING_SECURITY_USER_PASSWORD: suPerAdmin
      KAFKA_CLUSTERS_0_NAME: 10.10.0.106-kafka
      KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS: kafka1:9092,kafka2:9093,kafka3:9094
      KAFKA_CLUSTERS_0_PROPERTIES_SECURITY_PROTOCOL: SASL_PLAINTEXT
      KAFKA_CLUSTERS_0_PROPERTIES_SASL_MECHANISM: PLAIN
      KAFKA_CLUSTERS_0_PROPERTIES_SASL_JAAS_CONFIG: 'org.apache.kafka.common.security.plain.PlainLoginModule required username="admin" password="admin";'
```

相关命令：

```sh
# 启动
docker-compose up -d

# 移除
docker-compose down -v
```

## OffsetExplorer连接

![Snipaste_2022-09-27_14-19-28.png](/img1/Snipaste_2022-09-27_14-19-28.png)

![Snipaste_2022-09-27_14-20-06.png](/img1/Snipaste_2022-09-27_14-20-06.png)

![Snipaste_2022-09-27_14-20-31.png](/img1/Snipaste_2022-09-27_14-20-31.png)

![Snipaste_2022-09-27_14-20-54.png](/img1/Snipaste_2022-09-27_14-20-54.png)

JAAS Config 配置如下：

```cnf
org.apache.kafka.common.security.plain.PlainLoginModule required
    username="admin"
    password="12345678";
```

## DotNet6链接Kafka

生产者：

```cs
// See https://aka.ms/new-console-template for more information
using Confluent.Kafka;
using System.Net;

var config = new ProducerConfig
{
    BootstrapServers = "10.10.0.106:9092,10.10.0.106:9093,10.10.0.106:9094",
    SaslUsername = "admin",
    SaslPassword = "12345678",
    SecurityProtocol = SecurityProtocol.SaslPlaintext,
    SaslMechanism = SaslMechanism.Plain,
    ClientId = Dns.GetHostName(),
};


using (var producer = new ProducerBuilder<Null, string>(config).Build())
{
    string topic = "test";
    for (int i = 0; i < 100; i++)
    {
        var msg = "message " + i;
        Console.WriteLine($"Send message:   value {msg}");
        var result = await producer.ProduceAsync(topic, new Message<Null, string> { Value = msg });
        Console.WriteLine($"Result: key {result.Key} value {result.Value} partition:{result.TopicPartition}");
        Thread.Sleep(500);
    }
}

Console.ReadLine();
```

消费者：

```cs
// See https://aka.ms/new-console-template for more information

using Confluent.Kafka;
using System.Net;

Console.WriteLine("Hello World kafka consumer !");

var config = new ConsumerConfig
{ 
    GroupId = "foo",
    AutoOffsetReset = AutoOffsetReset.Earliest,
    BootstrapServers = "10.10.0.106:9092,10.10.0.106:9093,10.10.0.106:9094",
    SaslUsername = "admin",
    SaslPassword = "12345678",
    SecurityProtocol = SecurityProtocol.SaslPlaintext,
    SaslMechanism = SaslMechanism.Plain,
    ClientId = Dns.GetHostName(),
};

var cancel = false;

using (var consumer = new ConsumerBuilder<Ignore, string>(config).Build())
{
    var topic = "test";
    consumer.Subscribe(topic);

    while (!cancel)
    {
        var consumeResult = consumer.Consume(CancellationToken.None);

        Console.WriteLine($"Consumer message: {consumeResult.Message.Value} topic: {consumeResult.Topic} Partition: {consumeResult.Partition}");
    }

    consumer.Close();
}

```

参考：

[给 Kafka 配置 SASL/PLAIN 认证](https://kyle.ai/blog/7631.html)

[kafka-ui DOCKER_COMPOSE](https://github.com/provectus/kafka-ui/blob/master/documentation/compose/DOCKER_COMPOSE.md)

[Docker搭建带SASL用户密码验证的Kafka](https://www.cnblogs.com/zhouj850/p/15630101.html)

[docker-compose配置带密码验证的kafka](https://www.cnblogs.com/zhouj850/p/15683605.html)

[zookeeper、kafka加SASL认证后offsetExplorer如何连接](https://www.cnblogs.com/zhouj850/p/16580982.html)

[Kafka的安全认证机制SASL/PLAINTEXT](https://www.cnblogs.com/ilovena/p/10123516.html)

[Kafka三种可视化监控管理工具Monitor/Manager/Eagle](https://cloud.tencent.com/developer/article/1667262)

[Kafka, dotnet and SASL_SSL](https://managing.blue/2019/09/15/kafka-dotnet-and-sasl_ssl/)