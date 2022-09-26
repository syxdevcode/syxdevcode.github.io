---
title: Ubuntu22 Docker Compose安装Redis哨兵
date: 2022-08-24 17:25:21
tags:
  - Docker Compose
  - Docker
  - Ubuntu
  - Linux
categories:
  - Redis
---

## 目录

```shell
root@pony:/lims/redis# tree
.
├── conf
│   ├── master.conf
│   ├── sentinel-1.conf
│   ├── sentinel-2.conf
│   ├── sentinel-3.conf
│   ├── slave1.conf
│   └── slave2.conf
├── docker-compose.yml
├── master
│   └── data
├── slave1
│   └── data
└── slave2
    └── data
```

注意：新建的目录需要使用 `chmod` 命令授权。

<!--more-->
## 移除 redis(可选)

```sh
apt-get autoremove --purge redis-server
apt-get autoremove --purge redis-sentinel
```

## 配置文件

### 主从配置

**loglevel 安装默认的设置为 verbose：**

* 1）debug：会打印出很多信息，适用于开发和测试阶段
* 2）verbose（冗长的）：包含很多不太有用的信息，但比debug要清晰一些
* 3）notice：适用于生产模式
* 4）warning : 警告信息

`master.conf` 配置如下：

```shell
# 使用守护进程模式，如果为 yes,docker 则一直处于重试状态
daemonize no
# 非保护模式，可以外网访问
#protected-mode no
timeout 300
databases 16
rdbcompression yes
# 学习开发，使用最大日志级别，能够看到最多的日志信息
loglevel notice
#bind 127.0.0.1
#bind 10.10.0.106
port 6379

##暂且禁用指令重命名
##rename-command
##开启AOF，禁用snapshot
appendonly yes
# masteruser "PonyLimsCopy_Slave"
masterauth PN4Hxgcm.

requirepass PN4Hxgcm.

maxclients 10000
maxmemory 2000mb
logfile /data/redis.log

replica-announce-ip 10.10.0.106
replica-announce-port 36379

#cluster-announce-ip 10.10.0.106
#cluster-announce-port 36379
#cluster-announce-bus-port 46379

replica-serve-stale-data yes
replica-read-only yes

repl-diskless-sync no
repl-diskless-sync-delay 5

repl-diskless-load disabled
repl-disable-tcp-nodelay no
```

`slave1.conf` 配置如下：

```shell
# 使用守护进程模式，如果为 yes,docker 则一直处于重试状态
daemonize no
# 非保护模式，可以外网访问
# protected-mode no
timeout 300
databases 16
rdbcompression yes
# 学习开发，使用最大日志级别，能够看到最多的日志信息
loglevel notice
#bind 127.0.0.1
port 6379

##暂且禁用指令重命名
##rename-command
##开启AOF，禁用snapshot
appendonly yes
replicaof redis-master 6379
# replicaof 10.10.0.106 36379
# masteruser PonyLimsCopy_Slave
masterauth PN4Hxgcm.
requirepass PN4Hxgcm.

maxclients 10000
maxmemory 2000mb
logfile /data/redis.log

replica-announce-ip 10.10.0.106
replica-announce-port 36380

#cluster-announce-ip 10.10.0.106
#cluster-announce-port 36379
#cluster-announce-bus-port 46379

replica-serve-stale-data yes
replica-read-only yes

repl-diskless-sync no
repl-diskless-sync-delay 5

repl-diskless-load disabled
repl-disable-tcp-nodelay no
```

`slave1.conf` 配置如下：

注意 `replica-announce-port` 端口的区别。

```shell
# 使用守护进程模式，如果为 yes,docker 则一直处于重试状态
daemonize no
# 非保护模式，可以外网访问
protected-mode no
timeout 300
databases 16
rdbcompression yes
# 学习开发，使用最大日志级别，能够看到最多的日志信息
loglevel notice
# bind 127.0.0.1
port 6379

##暂且禁用指令重命名
##rename-command
##开启AOF，禁用snapshot
appendonly yes
replicaof redis-master 6379
# replicaof 10.10.0.106 36379
# masteruser PonyLimsCopy_Slave
masterauth PN4Hxgcm.
requirepass PN4Hxgcm.

maxclients 10000
maxmemory 2000mb
logfile /data/redis.log

replica-announce-ip 10.10.0.106
replica-announce-port 36381

#cluster-announce-ip 10.10.0.106
#cluster-announce-port 36379
#cluster-announce-bus-port 46379

replica-serve-stale-data yes
replica-read-only yes

repl-diskless-sync no
repl-diskless-sync-delay 5

repl-diskless-load disabled
repl-disable-tcp-nodelay no
```

### sentinel(哨兵)配置

docker 部署内部端口可以是一样的，只改变对外映射的端口号即可。

`sentinel-1.conf` 配置如下：

```shell
port 26379
dir "/tmp"
sentinel announce-ip 10.10.0.106
sentinel announce-port 26379
sentinel monitor limsmaster 10.10.0.106 36379 2
requirepass Pd4Ervty3w6NmSB2Fm.
sentinel auth-pass limsmaster PN4Hxgcm.
sentinel down-after-milliseconds limsmaster 30000
sentinel failover-timeout limsmaster 180000
sentinel parallel-syncs limsmaster 2
```

`sentinel-2.conf` 配置如下：

```shell
port 26379
dir "/tmp"
sentinel announce-ip 10.10.0.106
sentinel announce-port 26380
sentinel monitor limsmaster 10.10.0.106 36379 2
requirepass Pd4Ervty3w6NmSB2Fm.
sentinel auth-pass limsmaster PN4Hxgcm.
sentinel down-after-milliseconds limsmaster 30000
sentinel failover-timeout limsmaster 180000
sentinel parallel-syncs limsmaster 2
```

`sentinel-3.conf` 配置如下：

```shell
port 26379
dir "/tmp"
sentinel announce-ip 10.10.0.106
sentinel announce-port 26381
sentinel monitor limsmaster 10.10.0.106 36379 2
requirepass Pd4Ervty3w6NmSB2Fm.
sentinel auth-pass limsmaster PN4Hxgcm.
sentinel down-after-milliseconds limsmaster 30000
sentinel failover-timeout limsmaster 180000
sentinel parallel-syncs limsmaster 2
```

## docker-compose 配置

新建 `docker-compose.yml` 文件。

```yml
version: "3.1"

networks:
  net0:

services:
  redis-master:
    image: redis:7.0.4
    container_name: redis-master
    restart: always
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - /lims/redis/conf/master.conf:/usr/local/etc/redis/redis.conf
      - /lims/redis/master/data:/data
    ports:
      - 36379:6379
    networks:
      net0:
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海
  redis-slave1:
    image: redis:7.0.4
    container_name: redis-slave1
    restart: always
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - /lims/redis/conf/slave1.conf:/usr/local/etc/redis/redis.conf
      - /lims/redis/slave1/data:/data
    ports:
      - 36380:6379
    networks:
      net0:
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海
  redis-slave2:
    image: redis:7.0.4
    container_name: redis-slave2
    restart: always
    command: redis-server /usr/local/etc/redis/redis.conf
    volumes:
      - /lims/redis/conf/slave2.conf:/usr/local/etc/redis/redis.conf
      - /lims/redis/slave2/data:/data
    ports:
      - 36381:6379
    networks:
      net0:
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海
  redis-sentinel-1:
    image: redis:7.0.4
    container_name: redis-sentinel-1
    restart: always
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - /lims/redis/conf/sentinel-1.conf:/usr/local/etc/redis/sentinel.conf
      - /lims/redis/conf:/usr/local/etc/redis/conf/
    ports:
      - 26379:26379
    networks:
      net0:
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海
    depends_on:
      - "redis-master"
      - "redis-slave1"
      - "redis-slave2"
  redis-sentinel-2:
    image: redis:7.0.4
    container_name: redis-sentinel-2
    restart: always
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - /lims/redis/conf/sentinel-2.conf:/usr/local/etc/redis/sentinel.conf
      - /lims/redis/conf:/usr/local/etc/redis/conf/
    ports:
      - 26380:26379
    networks:
      net0:
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海
    depends_on:
      - "redis-master"
      - "redis-slave1"
      - "redis-slave2"
  redis-sentinel-3:
    image: redis:7.0.4
    container_name: redis-sentinel-3
    restart: always
    command: redis-sentinel /usr/local/etc/redis/sentinel.conf
    volumes:
      - /lims/redis/conf/sentinel-3.conf:/usr/local/etc/redis/sentinel.conf
      - /lims/redis/conf:/usr/local/etc/redis/conf/
    ports:
      - 26381:26379
    networks:
      net0:
    environment:
      - TZ=Asia/Shanghai # 时区配置亚洲上海
    depends_on:
      - "redis-master"
      - "redis-slave1"
      - "redis-slave2"
```

## docker-compose 命令

```shell
# 启动
docker compose up -d

# 移除
docker-compose down -v

# 查看日志
docker logs redis-master

# 停止
docker-compose stop

# 启动
docker-compose start

# 重启
docker-compose restart

# 暂停
docker-compose pause redis-master

# 取消暂停
docker-compose unpause redis-master
```

## 相关命令

```shell
docker exec -it redis-master redis-cli -a PN4Hxgcm.

docker exec redis-sentinel-1 redis-cli -h 10.10.0.106 -p 26379 -a PN4Hxgcm. SENTINEL get-master-addr-by-name limsmaster
docker exec redis-sentinel-2 redis-cli -h 10.10.0.106 -p 26379 -a PN4Hxgcm. SENTINEL get-master-addr-by-name limsmaster
docker exec redis-sentinel-3 redis-cli -h 10.10.0.106 -p 26379 -a PN4Hxgcm. SENTINEL get-master-addr-by-name limsmaster

docker exec redis-sentinel-1 redis-cli -h 10.10.0.106 -p 26379 -a PN4Hxgcm. sentinel masters
docker exec redis-sentinel-2 redis-cli -h 10.10.0.106 -p 26380 -a PN4Hxgcm. sentinel masters
docker exec redis-sentinel-3 redis-cli -h 10.10.0.106 -p 26381 -a PN4Hxgcm. sentinel masters

docker exec redis-sentinel-1 redis-cli -h 10.10.0.106 -p 26379 -a PN4Hxgcm. info sentinel
docker exec redis-sentinel-2 redis-cli -h 10.10.0.106 -p 26380 -a PN4Hxgcm. info sentinel
docker exec redis-sentinel-3 redis-cli -h 10.10.0.106 -p 26381 -a PN4Hxgcm. info sentinel

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis-master
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis-slave1
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' redis-slave2
```
