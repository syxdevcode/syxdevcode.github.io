---
title: Redis-Cluster集群部署
date: 2020-08-03 15:14:41
tags:
- Linux
- CentOS7
- Redis
- 群集
- 安装部署
categories:
- Redis
---

## 目录结构

完成公共的 redis.conf 和一个端口号的，如 `redis-6379.conf`，其它端口号的配置文件基于一个修改后的端口号配置文件即可。

```sh
[root@host-192-125-30-59 redis-cluster]# ls
data  log  redis-6379.conf  redis-6380.conf  redis-6381.conf  redis.conf
```

```sh
# 创建目录
mkdir /usr/local/redis-cluster
cd /usr/local/redis-cluster
mkdir -p data log

cd /usr/local/redis-cluster/data
mkdir -p 6379 6380 6381

cd /usr/local/redis-cluster

# 生成共有配置文件
touch redis.conf

# 生成端口文件
touch redis-6379.conf

# 生成不同端口的配置文件
sed 's/6379/6380/g' redis-6379.conf > /usr/local/redis-cluster/redis-6380.conf
sed 's/6379/6381/g' redis-6379.conf > /usr/local/redis-cluster/redis-6381.conf

# 启动
/usr/local/bin/redis-server /usr/local/redis-cluster/redis-6379.conf
/usr/local/bin/redis-server /usr/local/redis-cluster/redis-6380.conf
/usr/local/bin/redis-server /usr/local/redis-cluster/redis-6381.conf
```

查看redis运行情况：

```sh
netstat -tunpl | grep 6379
netstat -tunpl | grep 6380
netstat -tunpl | grep 6381
ps aux | grep redis

[root@host-192-125-30-111 redis-cluster]# netstat -tunpl | grep 6379
tcp        0      0 0.0.0.0:16379           0.0.0.0:*               LISTEN      6184/redis-server * 
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      6184/redis-server * 
tcp6       0      0 :::16379                :::*                    LISTEN      6184/redis-server * 
tcp6       0      0 :::6379                 :::*                    LISTEN      6184/redis-server * 
[root@host-192-125-30-111 redis-cluster]# netstat -tunpl | grep 6380
tcp        0      0 0.0.0.0:16380           0.0.0.0:*               LISTEN      6189/redis-server * 
tcp        0      0 0.0.0.0:6380            0.0.0.0:*               LISTEN      6189/redis-server * 
tcp6       0      0 :::16380                :::*                    LISTEN      6189/redis-server * 
tcp6       0      0 :::6380                 :::*                    LISTEN      6189/redis-server * 
[root@host-192-125-30-111 redis-cluster]# netstat -tunpl | grep 6381
tcp        0      0 0.0.0.0:16381           0.0.0.0:*               LISTEN      6194/redis-server * 
tcp        0      0 0.0.0.0:6381            0.0.0.0:*               LISTEN      6194/redis-server * 
tcp6       0      0 :::16381                :::*                    LISTEN      6194/redis-server * 
tcp6       0      0 :::6381                 :::*                    LISTEN      6194/redis-server * 
[root@host-192-125-30-111 redis-cluster]# ps aux|grep redis
root      6184  0.0  0.0 156964  7724 ?        Ssl  10:38   0:00 /usr/local/bin/redis-server *:6379 [cluster]
root      6189  0.0  0.0 153892  7692 ?        Ssl  10:39   0:00 /usr/local/bin/redis-server *:6380 [cluster]
root      6194  0.0  0.0 153892  7692 ?        Ssl  10:39   0:00 /usr/local/bin/redis-server *:6381 [cluster]
root      6339  0.0  0.0 112724   992 pts/1    S+   10:42   0:00 grep --color=auto redis

```

## 配置redis

指定端口的配置文件 `redis-PORT.conf`；
该文件定义所有与端口相关的配置项，PORT需要替换为具体的端口，如6379

### 公共配置 ：redis.conf

```sh
# Redis configuration file example.
#
# Note that in order to read the configuration file, Redis must be
# started with the file path as first argument:
#
# ./redis-server /path/to/redis.conf

# Note on units: when memory size is needed, it is possible to specify
# it in the usual form of 1k 5GB 4M and so forth:
#
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
#
# units are case insensitive so 1GB 1Gb 1gB are all the same.

# 建议
# 1,引用公共的配置文件，建议为全路径值
# include redis.conf

# yes表示以集群方式运行，为no表示以非集群方式运行
cluster-enabled yes

# 日志级别，建议为notice，
# 另外注意redis是不会滚动日志文件的，每次写日志都是先打开日志文件再写日志再关闭方式
# debug
# verbose
# notice
# warning
loglevel notice

# 密码
requirepass password

# 可选
# 最大连接数
maxclients 10000

# 可选
# 客户端多长（秒）时间没发包过来关闭它，0表示永不关闭
timeout 0

# 集群中的节点最大不可用时长，在这个时长内，不会被判定为fail。
# 对于master节点，当不可用时长超过此值时，slave在延迟至少0.5秒后会发起选举
# 进行failover成为master。Redis集群的很多其它值与cluster-node-timeout有关。
cluster-node-timeout 15000

# 5.0 以下使用cluster-slave-validity-factor
# 如果设置为0，则slave总是尝试成为master，无论slave和master间的链接断开时间的长短。
# 如果是一个大于0的值，则最大可断开时长为：(cluster-slave-validity-factor * cluster-node-timeout)。
# 例如：当cluster-node-timeout值为5，cluster-slave-validity-factor值为10时，
# slave和master间的连接断开50秒内，slave不会尝试成为master。
cluster-replica-validity-factor 0

# 这个参数一定不能小于repl-ping-replica-period，
# 可以考虑为repl-ping-replica-period的3倍或更大。定义多长时间内均PING不通时，
# 判定心跳超时。对于redis集群，达到这个值并不会发生主从切换,主从何时切换由
# 参数cluster-node-timeout控制，只有master状态为fail后，它的slaves才能发起选举。
repl-timeout 10

# 5.0 以下使用 repl-ping-slave-period
# 定义slave多久（秒）ping一次master，如果超过repl-timeout指定的时长都没有收到响应，
# 则认为master挂了
repl-ping-replica-period 10

# 5.0 以下使用 slave-read-only
# slave是否只读
replica-read-only yes

# 5.0 以下使用 slave-serve-stale-data
# 当slave与master断开连接，slave是否继续提供服务
replica-serve-stale-data yes

# 5.0 以下使用 slave-priority
# slave权重值，当master挂掉，只有权重最大的slave接替master
replica-priority 100

# 4.0新增配置项，用于控制是否启用RDB-AOF混用，值为no表示关闭
aof-use-rdb-preamble yes

# 当同时写AOF或RDB，则redis启动时只会加载AOF，AOF包含了全量数据。
# 如果当队列使用，入队压力又很大，建议设置为no
appendonly yes

# 可取值everysec，其中no表示由系统自动，
# 当写压力很大时，建议设置为no，否则容易造成整个集群不可用
# appendfsync always
# appendfsync everysec
# appendfsync no
appendfsync everysec

# 以守护进程运行 Redis 实例
# 相关配置项pidfile
daemonize yes                          

# 3.2.0新增的配置项，默认值为yes，限制从其它机器登录Redis server，而只能从127.0.0.1登录。
protected-mode no

# 取值不能超过系统的/proc/sys/net/core/somaxconn
# 此参数是指：已完成三次握手的TCP连接队列，默认值511，
# 但是Linux系统内核参数socket最大连接的值默认是128，对应文件/proc/sys/net/core/somaxconn，
# 当系统并发量大且客户端连接缓慢时，应该将两个值进行参考设置。
tcp-backlog 511

# 设置自动rewite AOF文件（手工rewrite只需要调用命令BGREWRITEAOF）
auto-aof-rewrite-percentage 100

# 触发rewrite的AOF文件大小，只有大于此大小时才会触发rewrite
auto-aof-rewrite-min-size 64mb

# 子进程在做rewrite时，主进程不调用fsync（由内核默认调度）
no-appendfsync-on-rewrite no

# 如果因为磁盘故障等导致保存rdb失败，停止写操作，可设置为NO。
stop-writes-on-bgsave-error yes

# 为no表示有slots不可服务时其它slots仍然继续服务，建议值为no，以提供最高的可用性
cluster-require-full-coverage no

# 设置最大的内存，单位为字节 
# 26843545600(25GB)
# 19327352832(18GB)
# 注意，在 64bit 系统下，maxmemory 设置为 0 表示不限制 Redis 内存使用，
# 在 32bit 系统下，maxmemory 隐式不能超过 3GB。 
maxmemory 19327352832

# 设置达到最大内存时的淘汰策略
maxmemory-policy volatile-lru

# 设置master端的客户端缓存，三种：normal、replica和pubsub
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60

# 最少slave(replica)数，用来保证集群中不会有裸奔的master。当某个master节点的slave(replica)节点挂掉裸奔后，
# 会从其他富余的master节点分配一个slave(replica)节点过来，确保每个master节点都有至少一个slave(replica)节点，
# 不至于因为master节点挂掉而没有相应slave(replica)节点替换为master节点导致集群崩溃不可用。
cluster-migration-barrier 1

# 当slave(replica)失联时的，环形复制缓区大小，值越大可容忍更长的slave(replica)失联时长
repl-backlog-size 1mb

# slave失联的时长达到该值时，释放backlog缓冲区
# repl-backlog-ttl 3600

# 刷新快照（RDB）到磁盘的策略，根据实际调整值，
# “save 900 1”表示900秒后至少有1个key被修改才触发save操作，其它类推。
# 注意执行flushall命令也会产生RDB文件，不过是空文件。
# 如果不想生成RDB文件，可以将save全注释掉。
save 900 1
save 300 10
save 60 10000
```

### 禁止指定命令

KEYS命令很耗时，FLUSHDB和FLUSHALL命令可能导致误删除数据，所以线上环境最好禁止使用，可以在Redis配置文件增加如下配置：

```sh
rename-command KEYS ""
rename-command FLUSHDB ""
rename-command FLUSHALL ""
```

### 端口配置：redis-PORT.conf

注：PORT 需要改成具体端口；

```sh
# 1,引用公共的配置文件，建议为全路径值
include /usr/local/redis-cluster/redis.conf

# 必须
# 端口号
port PORT

# 必须
# 默认放在dir指定的目录，注意不能包含目录，纯文件名，为redis-server进程自动维护，不能手工修改
cluster-config-file nodes-PORT.conf

#  可选
#  只有当daemonize值为yes时，才有意义；并且这个要求对目录/var/run有写权限，否则可以考虑设置为/#  tmp/redis-PORT.pid，或者放在bin或log目录下，如：/data/redis/log/redis-PORT.pid。只有当配#  置项daemonize的值为yes时，才会产生这个文件。
pidfile /var/run/redis-PORT.pid

# 工作目录。
#
# DB 将写入此目录，并指定文件名
# 以上使用"dbfilename"配置指令。
#
# 仅追加文件也将在此目录中创建。
#
# 请注意，您必须在此处指定目录，而不是文件名。
dir /usr/local/redis-cluster/data/PORT

# 纯文件名，位于dir指定的目录下，不能包含目录，
# 否则报错“appendfilename can't be a path, just # a filename”。
# 如果开启了AOF，REdis进程启动时并不会读取RDB文件，
# 所以配置上可以考虑关闭RDB，这样可以提升REdis稳定性。
dbfilename dump-PORT.rdb

# 纯文件名，位于dir指定的目录下，不能包含目录，
# 否则报错“appendfilename can't be a path, just a filename”
appendfilename "appendonly-PORT.aof"

# 日志文件，包含目录和文件名，注意redis不会自动滚动日志文件
logfile /usr/local/redis-cluster/log/redis-PORT.log
```

## 创建和启动redis集群

### 快速创建,启动redis集群

如果只是想快速创建和启动redis集群，而不关心过程，可使用redis官方提供的脚本 `create-cluster`，两步完成：

```sh
create-cluster start
create-cluster create
```

第二步 `create-cluster create` 是一个交互式过程，当提示时，请输入 `yes` 再回车继续，第一个节点的端口号为 30001，一共会启动六个redis节点。

create-cluster位于redis源代码的 `utils/create-cluster` 目录下，是一个bash脚本文件。

停止集群：`create-cluster stop`。

### 创建redis cluster

```sh
# 注：由于只有两台服务器，故在服务器1上部署两个主节点，一个从节点，服务器2上部署两个从节点，一个主节点。
/usr/local/bin/redis-cli --cluster create 192.125.30.111:6379 192.125.30.111:6380 192.125.30.59:6381 192.125.30.59:6379 192.125.30.59:6380 192.125.30.111:6381 --cluster-replicas 1
```

`redis-cli` 的参数说明：

```sh
# 表示创建一个redis集群。
create

# 表示为集群中的每一个主节点指定一个从节点，即一比一的复制。
--cluster-replicas 1
```

注意：如果配置项 `cluster-enabled` 的值不为yes，则执行时会报错 `[ERR] Node 192.168.0.251:6381 is not configured as a cluster node.`

**ps aux|grep redis-server**

查看redis进程是否已切换为集群状态（cluster）

停止redis实例，直接使用kill命令即可，如：kill 3825，重启和单机版相同。

**从slaves读数据**

默认不能从slaves读取数据，但建立连接后，执行一次命令 `READONLY` ，即可从slaves读取数据。如果想再次恢复不能从slaves读取数据，可以执行下命令 `READWRITE`。

## 群集操作

### 新增主（master）节点


注意一定要保证新节点里面没有添加过任何数据

以添加 `192.168.0.251:6390` 为例：

```sh
# 192.168.0.251:6381 为集群中任一可用的节点
redis-cli --cluster add-node 192.168.0.251:6390 192.168.0.251:6381

# 查看
redis-cli -c -p 6381 cluster nodes|grep master
```

新加入的master节点上没有任何数据（slots，运行redis命令cluster nodes可以看到这个情况）。当一个slave想成为master时，由于这个新的master节点不管理任何slots，它不参与选举。可以使用redis-cli的reshard为这个新master节点分配slots，如：

```sh
redis-cli --cluster reshard 192.168.0.251:6390
```

### 新增从（slave）节点

以添加 `192.168.0.251:6390` 为例：

```sh
redis-cli --cluster add-node 192.168.0.251:6390 192.168.0.251:6381 --cluster-slave
```

`192.168.0.251:6390` 为新添加的从节点，`192.168.0.251:6381` 可为集群中已有的任意节点，这种方法随机为6390指定一个master，如果想明确指定master，假设目标master的ID为`3c3a0c74aae0b56170ccb03a76b60cfe7dc1912e`，则：

```sh
redis-cli --cluster add-node 192.168.0.251:6390 192.168.0.251:6381 --cluster-slave --cluster-master-id 3c3a0c74aae0b56170ccb03a76b60cfe7dc1912e
```

### 删除节点

从集群中删除一个节点命令格式：

```sh
# 127.0.0.1:7000 为集群中任意一个非待删除节点，node-id 为待删除节点的ID。
redis-cli --cluster del-node 127.0.0.1:7000 `<node-id>`
```

如果待删除的是master节点，则在删除之前需要将该master负责的slots先全部迁到其它master。

$ ./redis-cli --cluster del-node 192.168.0.251:6381 082c079149a9915612d21cca8e08c831a4edeade

>>> Removing node 082c079149a9915612d21cca8e08c831a4edeade from cluster 192.168.0.251:6381

>>> Sending CLUSTER FORGET messages to the cluster...

>>> SHUTDOWN the node.


如果删除后，其它节点还看得到这个被删除的节点，则可通过FORGET命令解决，需要在所有还看得到的其它节点上执行：

CLUSTER FORGET `<node-id>`

FORGET做两件事：

1) 从节点表剔除节点；

2) 在60秒的时间内，阻止相同ID的节点加进来。

### slots相关命令

```sh
CLUSTER ADDSLOTS slot1 [slot2] ... [slotN]
CLUSTER DELSLOTS slot1 [slot2] ... [slotN]
CLUSTER SETSLOT slot NODE node
CLUSTER SETSLOT slot MIGRATING node
CLUSTER SETSLOT slot IMPORTING node
```

参考：

[Redis-5.0.0集群配置](https://blog.csdn.net/Aquester/article/details/90511200)

[分布式缓存 Redis 集群搭建](https://www.cnblogs.com/esofar/p/10486621.html)

[redis集群配置文件](https://www.cnblogs.com/zh-dream/p/12275031.html)

[Redis详解（二）-- redis的配置文件介绍](https://www.cnblogs.com/ysocean/p/9074787.html)

[高可用Redis(十一)：使用redis-trib.rb工具搭建集群](https://www.cnblogs.com/renpingsheng/p/9833740.html)

[Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)

[Redis Cluster Specification](https://redis.io/topics/cluster-spec)