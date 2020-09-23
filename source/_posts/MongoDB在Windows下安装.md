---
title: MongoDB在Windows下安装
date: 2020-09-21 15:59:13
tags:
- Windows
- MongoDB
- 安装部署
categories:
- MongoDB
---

## 下载安装包,安装

[https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)

默认安装位置：`C:\Program Files\MongoDB\Server\4.4\bin`

## 运行 MongoDB 服务器

**命令行运行**

```sh
cd C:\Program Files\MongoDB\Server\4.4\bin\
mongod --dbpath D:\MongoDBData
```

![微信截图_20200921162755.png](/img/微信截图_20200921162755.png)

**连接服务**

```sh
cd C:\Program Files\MongoDB\Server\4.4\bin\
mongo.exe
```

![微信截图_20200921162957.png](/img/微信截图_20200921162957.png)

## 配置 MongoDB 服务

`mongod` 默认配置文件：

![微信截图_20200921161709.png](/img/微信截图_20200921161709.png)

根据实际情况，修改日志路径，和文档路径。

如：`dbPath: D:\MongoDBData`

```conf
# mongod.conf

# for documentation of all options, see:
#   http://docs.mongodb.org/manual/reference/configuration-options/

# Where and how to store data.
storage:
  dbPath: C:\Program Files\MongoDB\Server\4.4\data
  journal:
    enabled: true
#  engine:
#  mmapv1:
#  wiredTiger:

# where to write logging data.
systemLog:
  destination: file
  logAppend: true
  path:  C:\Program Files\MongoDB\Server\4.4\log\mongod.log

# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1


#processManagement:

#security:

#operationProfiling:

#replication:

#sharding:

## Enterprise-Only Options:

#auditLog:

#snmp:
```

## 安装 MongoDB服务

```sh
mongod.exe --config "C:\Program Files\MongoDB\Server\4.4\bin\mongod.cfg" --install
```

dbpath，可以在配置文件（例如：C:\mongodb\mongod.cfg）或命令行中通过 `--dbpath` 选项指定。

可以安装 `mongod.exe` 或 `mongos.exe` 的多个实例的服务。只需要通过使用 `--serviceName` 和 `--serviceDisplayName` 指定不同的实例名。


以管理员身份启动 cmd：

```sh
# 启动MongoDB服务
net start MongoDB

# 关闭MongoDB服务
net stop MongoDB

# 移除 MongoDB 服务
cd C:\Program Files\MongoDB\Server\4.4\bin
mongod.exe --remove
```

## MongoDB 后台管理 Shell

打开 `mongodb` 装目录的下的bin目录，然后执行 `mongo.exe` 文件，`MongoDB Shell` 是`MongoDB` 自带的交互式 `Javascript shell` ,用来对MongoDB进行操作和管理的交互式环境。

当进入mongoDB后台后，它默认会链接到 test 文档（数据库）：

```sh
C:\Program Files\MongoDB\Server\4.4\bin>mongo
MongoDB shell version v4.4.1
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("7aa2a8eb-097a-4433-a348-314868079dff") }
MongoDB server version: 4.4.1
---
The server generated these startup warnings when booting:
        2020-09-21T16:38:42.622+08:00: ***** SERVER RESTARTED *****
        2020-09-21T16:38:43.487+08:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
---
---
        Enable MongoDB's free cloud-based monitoring service, which will then receive and display
        metrics about your deployment (disk utilization, CPU, operation statistics, etc).

        The monitoring data will be available on a MongoDB website with a unique URL accessible to you
        and anyone you share the URL with. MongoDB may use this information to make product
        improvements and to suggest MongoDB products and deployment options to you.

        To enable free monitoring, run the following command: db.enableFreeMonitoring()
        To permanently disable this reminder, run the following command: db.disableFreeMonitoring()
---
> db
test
> 2 + 2
4
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

参考：

[Windows 平台安装 MongoDB](https://www.runoob.com/mongodb/mongodb-window-install.html)