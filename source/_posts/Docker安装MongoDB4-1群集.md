---
title: Docker安装MongoDB4.1群集
date: 2018-08-21 10:37:53
tags:
- Linux
- MongoDB
- CentOS7
- 安装部署
- 分布式
- Docker Compose
- Docker
categories: 
- MongoDB
---
# docker安装mongodb4.1群集

官网：[https://www.mongodb.com](https://www.mongodb.com)

镜像：[https://hub.docker.com/_/mongo/](https://hub.docker.com/_/mongo/)

github: [Dockerfile](https://github.com/docker-library/mongo/blob/4958ee0a8a6b1ac1c3734b161db67371720b31b5/4.1/Dockerfile)

## 相关概念

![sharded-cluster-production-architecture.bakedsvg.svg](/img/sharded-cluster-production-architecture.bakedsvg.svg)

路由，分片、副本集、配置服务器

四个组件：mongos、config server、shard、replica set。

`mongos`，数据库集群请求的入口，所有的请求都通过mongos进行协调，不需要在应用程序添加一个路由选择器，mongos自己就是一个请求分发中心，它负责把对应的数据请求请求转发到对应的shard服务器上。在生产环境通常有多mongos作为请求的入口，防止其中一个挂掉所有的mongodb请求都没有办法操作。

`config server`，顾名思义为配置服务器，存储所有数据库元信息（路由、分片）的配置。mongos本身没有物理存储分片服务器和数据路由信息，只是缓存在内存里，配置服务器则实际存储这些数据。mongos第一次启动或者关掉重启就会从 config server 加载配置信息，以后如果配置服务器信息变化会通知到所有的 mongos 更新自己的状态，这样 mongos 就能继续准确路由。在生产环境通常有多个 config server 配置服务器，因为它存储了分片路由的元数据，防止数据丢失！

`shard`，分片（sharding）是指将数据库拆分，将其分散在不同的机器上的过程。将数据分散到不同的机器上，不需要功能强大的服务器就可以存储更多的数据和处理更大的负载。基本思想就是将集合切成小块，这些块分散到若干片里，每个片只负责总数据的一部分，最后通过一个均衡器来对各个分片进行均衡（数据迁移）。

`replica set`，中文翻译副本集，其实就是shard的备份，防止shard挂掉之后数据丢失。复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。

`仲裁者（Arbiter）`，是复制集中的一个MongoDB实例，它并不保存数据。仲裁节点使用最小的资源并且不要求硬件设备，不能将Arbiter部署在同一个数据集节点中，可以部署在其他应用服务器或者监视服务器中，也可部署在单独的虚拟机中。为了确保复制集中有奇数的投票成员（包括primary），需要添加仲裁节点做为投票，否则primary不能运行时不会自动切换primary。

注：mongodb3.4以后要求配置服务器也创建副本集，不然集群搭建不成功。

## 部署

目录结构：

```bash
mongodb
 |--docker-compose.yml
 |- mongodbtest
```

创建2个分片服务（shardsvr），每个shardsvr包含3个副本，其中1个主节点，1个从节点，1个仲裁节点。

创建3个配置服务（configsvr）

创建2个路由服务（mongos） 命令：configdb

### 创建目录

在`mongodb`目录运行以下命令生成目录：

```bash
mkdir -p /home/toor/Documents/mongodb/mongodbtest/cs/rs1 /home/toor/Documents/mongodb/mongodbtest/cs/rs2 /home/toor/Documents/mongodb/mongodbtest/cs/rs3 # config server replset
mkdir -p /home/toor/Documents/mongodb/mongodbtest/sh1/rs1 /home/toor/Documents/mongodb/mongodbtest/sh1/rs2 /home/toor/Documents/mongodb/mongodbtest/sh1/rs3 # sharding
mkdir -p /home/toor/Documents/mongodb/mongodbtest/sh2/rs1 /home/toor/Documents/mongodb/mongodbtest/sh2/rs2 /home/toor/Documents/mongodb/mongodbtest/sh2/rs3 # sharding
mkdir -p /home/toor/Documents/mongodb/mongodbtest/mongos1
mkdir -p /home/toor/Documents/mongodb/mongodbtest/mongos2
```

### 创建docker-compose

```bash
touch docker-compose.yml
vim docker-compose.yml
```

docker-compose.yml内容：

```bash
version: '3'
services:
  csrs1:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/cs/rs1:/data/db
    command: mongod --noauth --configsvr --replSet csrs --dbpath /data/db
  csrs2:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/cs/rs2:/data/db
    command: mongod --noauth --configsvr --replSet csrs --dbpath /data/db
  csrs3:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/cs/rs3:/data/db
    command: mongod --noauth --configsvr --replSet csrs --dbpath /data/db
  mongos1:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/mongos1:/data/db
    command: mongos --noauth --configdb csrs/csrs1:27019,csrs2:27019,csrs3:27019
  mongos2:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/mongos2:/data/db
    command: mongos --noauth --configdb csrs/csrs1:27019,csrs2:27019,csrs3:27019
  sh1rs1:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/sh1/rs1:/data/db
    command: mongod --noauth --dbpath /data/db --shardsvr --replSet shrs1
  sh1rs2:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/sh1/rs2:/data/db
    command: mongod --noauth --dbpath /data/db --shardsvr --replSet shrs1
  sh1rs3:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/sh1/rs3:/data/db
    command: mongod --noauth --dbpath /data/db --shardsvr --replSet shrs1
  sh2rs1:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/sh2/rs1:/data/db
    command: mongod --noauth --dbpath /data/db --shardsvr --replSet shrs2
  sh2rs2:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/sh2/rs2:/data/db
    command: mongod --noauth --dbpath /data/db --shardsvr --replSet shrs2
  sh2rs3:
    image: mongo:latest
    volumes:
      - $PWD/mongodbtest/sh2/rs3:/data/db
    command: mongod --noauth --dbpath /data/db --shardsvr --replSet shrs2
```

注：

```bash
--noauth 不进行身份认证

--auth 进行身份认证

--nohttpinterface
不加会有一个28017的端口监听，可以通过网页管理mongodb，不需要请去掉

--bind_ip
可以限制访问的ip

--port
可以重新制定端口，默认为27017
```

### 初始化[配置服务]副本集(Config Server)

```bash
docker-compose exec csrs1 mongo --port 27019
rs.initiate()
rs.add('csrs2:27019')
rs.add('csrs3:27019')
rs.status() //查看状态
quit() // 退出 或 exit
```

注：config server 默认端口为 27019

### 初始化分片

注：'shard server' 默认端口号为 27018

#### shrs1副本集

```bash
docker-compose exec sh1rs1 mongo --port 27018
rs.initiate()
var cfg = rs.conf()
cfg.members[0].host = 'sh1rs1:27018'
rs.reconfig(cfg)
rs.add('sh1rs2:27018')
rs.add({host:"sh1rs3:27018", arbiterOnly:true}) // 仲裁
quit()
```

#### shrs2副本集

```bash
docker-compose exec sh2rs1 mongo --port 27018
rs.initiate()
var cfg = rs.conf()
cfg.members[0].host = 'sh2rs1:27018'
rs.reconfig(cfg)
rs.add('sh2rs2:27018')
rs.add({host:"sh2rs3:27018", arbiterOnly:true})
quit()
```

### 通过mongos添加分片关系到configsvr

mongos1:

```bash
docker-compose exec mongos1 mongo
sh.addShard('shrs1/sh1rs1:27018,sh1rs2:27018,sh1rs3:27018')
sh.addShard('shrs2/sh2rs1:27018,sh2rs2:27018,sh2rs3:27018')
sh.status() //查看状态
```

mongos2:

```bash
docker-compose exec mongos2 mongo
sh.addShard('shrs1/sh1rs1:27018,sh1rs2:27018,sh1rs3:27018')
sh.addShard('shrs2/sh2rs1:27018,sh2rs2:27018,sh2rs3:27018')
sh.status() //查看状态
```

![QQ截图20180824094655.png](/img/QQ截图20180824094655.png)

## 启用分片

### 设置分片chunk大小

```bash
use config
db.settings.save({"_id":"chunksize","value":1}) // //设置块大小为1M是方便实验，不然就需要插入海量数据
```

### 启用数据库分片

```bash
// 数据库分片就有针对性，可以自定义需要分片的库或者表，毕竟也不是所有数据都是需要分片操作的
sh.enableSharding("python")

// 为表创建的索引,以”id“为索引
db.user.createIndex({"id":1})

// 启用表分片
sh.shardCollection("python.user",{"id":1})
sh.status()
```

### 模拟写入数据

```bash
use python
for(i=1;i<=50000;i++){db.user.insert({"id":i,"name":"jack"+i})}
sh.status()
db.user.stats()
db.user.count()
show dbs
```

## 后期运维

mongodb的启动顺序是，先启动配置服务器，在启动分片，最后启动mongos.

关闭时，直接killall杀掉所有进程

例：

```bash
killall mongod
killall mongos
```

参考：

[mongodb 3.4 集群搭建：分片+副本集](http://www.ityouknow.com/mongodb/2017/08/05/mongodb-cluster-setup.html)

[MongoDB(4.0)分片——大数据的处理之道](http://blog.51cto.com/13643643/2148825)

[mongodb 3.4 集群搭建升级版 五台集群](https://www.cnblogs.com/ityouknow/p/7566682.html)

[docker 搭建 mongo 集群](https://www.jianshu.com/p/4d86e9c33e0d)

[[原创]在Docker上部署mongodb分片副本集群](https://www.cnblogs.com/hehexiaoxia/p/6192796.html)