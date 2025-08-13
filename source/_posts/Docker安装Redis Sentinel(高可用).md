---
title: Docker安装Redis Sentinel(高可用)
date: 2018-05-21 08:51:08
tags:
- Redis
- Docker
- 安装部署
categories: 
- Redis
---
# Docker安装redis Sentinel(高可用)

## redis介绍

redis集群有两种，一种是redis sentinel(哨兵)，高可用集群，同时只有一个master，各实例数据保持一致；一种是redis cluster，分布式集群，同时有多个master，数据分片部署在各个master上。 哨兵模式简单说就是在后台有一个监控，监控当前的主机并巡逻主机下面的从机，如果某一时刻主机挂掉了，那么他会通过一种投票的机制从从机之中选举一台作为新的主机，并且其余的从机将会连接到这个新的主机上面，完成故障转移,主从模式就是一个中心化的结构,通过哨兵完成去中心化；本次要搭建的是redis sentinel。

Redis 的 Sentinel 系统用于管理多个 Redis 服务器（instance）， 该系统执行以下三个任务：

* 监控（Monitoring）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
* 提醒（Notification）： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
* 自动故障迁移（Automatic failover）： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

## 目录结构

``` redis
redis
 |--master
 |--slave1
 |--slave2
 |--Dockerfile
 |--redis.conf
 |--docker-compose.yml
 |--sentinel.conf
 |--start.sh
```

## 制作镜像

整个集群可以分为一个master，N个slave，M个sentinel，本次以2个slave和3个sentinel为例

### 增加redis.conf

```conf
# 使用守护进程模式
daemonize no
# 非保护模式，可以外网访问
protected-mode no
timeout 300
databases 16
rdbcompression yes
# 学习开发，使用最大日志级别，能够看到最多的日志信息
loglevel debug
port $redis_port
##授权密码，各个配置保持一致
##暂且禁用指令重命名
##rename-command
##开启AOF，禁用snapshot
appendonly yes
#slaveof redis-master $master_port
slave-read-only yes
maxclients 10000
maxmemory 1000mb
```

### 增加sentinel.conf

```conf
port $sentinel_port
dir "/tmp"
sentinel monitor mymaster redis-master $master_port 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout manager1 180000
sentinel parallel-syncs manager1 2
```

这四行配置为一组，因为我们只有一个 master 节点，所以只配置了一个 master，可以配置多个 master，不用配置 slave 的信息，因为 slave 能够被自动检测到（master 节点会有关于 slave 的消息）

其他选项的基本格式如下：

sentinel <选项的名字> <主服务器的名字> <选项的值>

* `auth-pass`：选项指定了 master 的连接密码。
* `down-after-milliseconds`：选项指定了 Sentinel 认为服务器已经断线所需的毫秒数。
* `failover-timeout`：如果在该时间（ms）内未能完成 failover 操作，则认为该 failover 失败。
* `parallel-syncs`：选项指定了在执行故障转移时，最多可以有多少个从服务器同时对新的主服务器进行同步，这个数字越小，完成故障转移所需的时间就越长。

### 添加start.sh

``` conf
cd /conf
redis_role=$1
echo $redis_role
if [ $redis_role = "master" ] ; then
    echo "master"
    sed -i "s/\$redis_port/$redis_port/g" redis.conf
    redis-server /conf/redis.conf
elif [ $redis_role = "slave" ] ; then
    echo "slave"
    sed -i "s/\$redis_port/$redis_port/g" redis.conf
    sed -i "s/#slaveof/slaveof/g" redis.conf
    sed -i "s/\$master_port/$master_port/g" redis.conf
    redis-server /conf/redis.conf
elif [ $redis_role = "sentinel" ] ; then
    echo "sentinel"
    sed -i "s/\$sentinel_port/$sentinel_port/g" sentinel.conf
    sed -i "s/\$master_port/$master_port/g" sentinel.conf
    redis-sentinel /conf/sentinel.conf
else
    echo "unknow role!"
fi     #ifend
```

### 添加Dockerfile

``` docker
FROM redis:4-alpine

# 设置时区
RUN apk add --update tzdata
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone


COPY redis.conf /conf/redis.conf
COPY sentinel.conf /conf/sentinel.conf
COPY start.sh /conf/start.sh
RUN chmod +x /conf/start.sh && chown redis:redis /conf/*
ENTRYPOINT ["sh","/conf/start.sh"]
CMD ["master"]
```

### build镜像

在当前目录执行以下命令，创建redis:v1镜像。

```docker
docker build -t redis:v1 .
```

## 启动redis服务

在根目录创建`master`,`slave1`,`slave2`目录。

创建`docker-compose.yml`

```docker
version: '3'

services:
  redis-master:
    container_name: redis-master
    build:
      context: ./master
    volumes:
      - /master/data:/data
    environment:
      redis_port: '16379'
    image: redis:v1
    expose:
      - "16379"
  redis-slave1:
    container_name: redis-slave1
    build:
      context:./slave1
    volumes:
      - /slave1/data:/data
    environment:
      master_port: '16379'
      redis_port: '16380'
    expose:
      - "16380"
    command:
      - slave
    image: redis:v1
  redis-slave2:
    container_name: redis-slave2
    build:
      context:./slave2
    volumes:
      - /slave2/data:/data
    environment:
      master_port: '16379'
      redis_port: '16380'
    expose:
      - "16380"
    command:
      - slave
    image: redis:v1
  redis-sentinel-1:
    container_name: redis-sentinel-1
    environment:
      master_port: '16379'
      sentinel_port: '16381'
    expose:
      - "16381"
    command:
      - sentinel
    depends_on:
      - "redis-master"
      - "redis-slave1"
      - "redis-slave2"
    image: redis:v1
  redis-sentinel-2:
    container_name: redis-sentinel-2
    environment:
      master_port: '16379'
      sentinel_port: '16381'
    expose:
      - "16381"
    command:
      - sentinel
    depends_on:
      - "redis-master"
      - "redis-slave1"
      - "redis-slave2"
    image: redis:v1
  redis-sentinel-3:
    container_name: redis-sentinel-3
    environment:
      master_port: '16379'
      sentinel_port: '16381'
    expose:
      - "16381"
    command:
      - sentinel
    depends_on:
      - "redis-master"
      - "redis-slave1"
      - "redis-slave2"
    image: redis:v1
```

运行以下命令以启动：

```docker
docker-compose up
```

可以选择添加`-d`参数以守护进程启动。

## Redis Sentinel 故障转移测试

### 查看redis-master服务IP

```docker
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis-master
```

结果：

```docker
[root@localhost redis]# docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis-master
172.18.0.2
```

### 查看mymaster节点IP

```docker
docker exec redis-sentinel-1 redis-cli -p 16381 SENTINEL get-master-addr-by-name mymaster
```

结果：

```docker
[root@localhost redis]# docker exec redis-sentinel-1 redis-cli -p 16381 SENTINEL get-master-addr-by-name mymaster
172.18.0.2
16379
```

### 连接client

```docker
docker exec -it redis-master redis-cli -h 172.18.0.2 -p 16379
```

### 停止master节点

```docker
docker pause redis-master
```

### 查询master主节点IP

注意：中间有时间间隔。

```docker
docker exec redis-sentinel-1 redis-cli -p 16381 SENTINEL get-master-addr-by-name mymaster
```

结果：

```docker
[root@localhost redis]# docker exec redis-sentinel-1 redis-cli -p 16381 SENTINEL get-master-addr-by-name mymaster
172.18.0.4
16380
```

### 恢复redis-master服务

```docker
docker unpause redis-master
```

### 分别查看slave1，slave2服务IP

```docker
[root@localhost redis]# docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis-slave1
172.18.0.4
[root@localhost redis]# docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis-slave2
172.18.0.3
```

### 停止主节点

```docker
docker pause redis-slave1
```

### 查询主节点

```docker
docker exec redis-sentinel-1 redis-cli -p 16381 SENTINEL get-master-addr-by-name mymaster
```

结果：

```docker
localhost redis]# docker exec redis-sentinel-1 redis-cli -p 16381 SENTINEL get-master-addr-by-name mymaster
172.18.0.2
16379
```

至此，故障转移测试完成。

参考：

[高可用Redis服务架构分析与搭建](https://www.cnblogs.com/xuning/p/8464625.html)

[Redis Sentinel 高可用服务架构搭建](https://www.cnblogs.com/xishuai/p/redis-sentinel.html)

[Redis哨兵-实现Redis高可用](http://redis.majunwei.com/topics/sentinel.html)

[Redis Sentinel机制与用法（一）](https://segmentfault.com/a/1190000002680804)

[Redis设置认证密码 Redis使用认证密码登录 在Redis集群中使用认证密码](https://itbilu.com/database/redis/Ey_r7mWR.html)

[Docker化高可用redis集群](https://yq.aliyun.com/articles/58003)

[Docker环境搭建redis集群(主从模式)](https://www.jianshu.com/p/a63a98fa047d)

[https://github.com/AliyunContainerService/redis-cluster/blob/master/test.sh](https://github.com/AliyunContainerService/redis-cluster/blob/master/test.sh)

[Redis 的 Sentinel 文档](http://www.redis.cn/topics/sentinel.html?spm=a2c4e.11153940.blogcont58003.8.5e0d6772hFOEsg)