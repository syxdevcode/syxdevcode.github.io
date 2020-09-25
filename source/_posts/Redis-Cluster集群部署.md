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

## 修改系统参数

### 修改最大可打开文件数

参考：[Linux最大文件打开数 ](https://syxdevcode.github.io/2020/08/04/Linux%E6%9C%80%E5%A4%A7%E6%96%87%E4%BB%B6%E6%89%93%E5%BC%80%E6%95%B0/)

### TCP监听队列大小

参考：[Linux设置TCP监听队列](https://syxdevcode.github.io/2020/08/04/Linux%E8%AE%BE%E7%BD%AETCP%E7%9B%91%E5%90%AC%E9%98%9F%E5%88%97/)

### OOM相关：vm.overcommit_memory

参考：[Linux OOM机制简介](https://syxdevcode.github.io/2020/08/05/Linux%20OOM%E6%9C%BA%E5%88%B6%E7%AE%80%E4%BB%8B/)

### Linux Transparent HugePages(透明大页)

参考：[Linux Transparent HugePages(透明大页)](https://syxdevcode.github.io/2020/08/05/Linux%20Transparent%20HugePages(%E9%80%8F%E6%98%8E%E5%A4%A7%E9%A1%B5)/)

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
maxclients 100000

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

### 加上进程监控

使用 [https://github.com/eyjian/libmooon/blob/master/shell/process_monitor.sh](https://github.com/eyjian/libmooon/blob/master/shell/process_monitor.sh)

监控示例：

```sh
REDIS_HOME=/usr/local
* * * * * /usr/local/bin/process_monitor.sh "$REDIS_HOME/bin/redis-server 6379" "$REDIS_HOME/bin/redis-server $REDIS_HOME/redis-cluster/redis-6379.conf"
* * * * * /usr/local/bin/process_monitor.sh "$REDIS_HOME/bin/redis-server 6380" "$REDIS_HOME/bin/redis-server $REDIS_HOME/redis-cluster/redis-6380.conf"
* * * * * /usr/local/bin/process_monitor.sh "$REDIS_HOME/bin/redis-server 6381" "$REDIS_HOME/bin/redis-server $REDIS_HOME/redis-cluster/redis-6381.conf"
```

注意：redis的日志文件不会自动滚动，`redis-server` 每次在写日志时，均会以追加方式调用fopen写日志，而不处理滚动。
也可借助linux自带的 `logrotate` 来滚动redis日志，命令 `logrotate` 一般位于目录 `/usr/sbin` 下。

滚动日志（可选）:

```sh
* * * * * log=$REDIS_HOME/redis-cluster/log/redis-6379.log;if test `ls -l $log|cut -d' ' -f5` -gt 104857600; then mv $log $log.old; fi
* * * * * log=$REDIS_HOME/redis-cluster/log/redis-6380.log;if test `ls -l $log|cut -d' ' -f5` -gt 104857600; then mv $log $log.old; fi
* * * * * log=$REDIS_HOME/redis-cluster/log/redis-6381.log;if test `ls -l $log|cut -d' ' -f5` -gt 104857600; then mv $log $log.old; fi
```

### Redis集群TCP端口（Redis Cluster TCP ports）

每个Redis集群中的节点都需要打开两个TCP连接。一个连接用于正常的给Client提供服务，比如6379，还有一个额外的端口（通过在这个端口号上加10000）作为数据端口，比如16379。第二个端口（本例中就是16379）用于集群总线，这是一个用二进制协议的点对点通信信道。这个集群总线（Cluster bus）用于节点的失败侦测、配置更新、故障转移授权，等等。客户端从来都不应该尝试和这些集群总线端口通信，它们只应该和正常的Redis命令端口进行通信。注意，确保在你的防火墙中开放着两个端口，否则，Redis集群节点之间将无法通信。

命令端口和集群总线端口的偏移量总是10000。

注意，如果想要集群按照你想的那样工作，那么集群中的每个节点应该：

正常的客户端通信端口（通常是6379）用于和所有可到达集群的所有客户端通信
集群总线端口（the client port + 10000）必须对所有的其它节点是可到达的
也就是，要想集群正常工作，集群中的每个节点需要做到以下两点：

正常的客户端通信端口（通常是6379）必须对所有的客户端都开放，换言之，所有的客户端都可以访问
集群总线端口（客户端通信端口 + 10000）必须对集群中的其它节点开放，换言之，其它任意节点都可以访问
如果你没有开放TCP端口，你的集群可能不会像你期望的那样工作。集群总线用一个不同的二进制协议通信，用于节点之间的数据交换。

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

如果群集设置着密码，需要使用 `-a 密码`，否则会报错：`(error)NOAUTH Authentication required`:

```sh
/usr/local/bin/redis-cli -a test --cluster create 192.125.30.111:6379 192.125.30.111:6380 192.125.30.59:6381 192.125.30.59:6379 192.125.30.59:6380 192.125.30.111:6381 --cluster-replicas 1
```

注意：如果报以下错误，需要停止单个redis节点，并把对应的数据目录删除重建。

```sh
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
[ERR] Node 192.125.30.59:6381 is not empty. Either the node already knows other nodes (check with CLUSTER NODES) or contains some key in database 0.
```

```sh
#停止节点，
[root@host-192-125-30-111 ~]# ps aux | grep redis
root     10547  0.0  0.0 112724   988 pts/0    R+   11:03   0:00 grep --color=auto redis
root     17367  0.1  0.0 156452  7920 ?        Ssl  8月15   3:56 /usr/local/bin/redis-server *:6379 [cluster]
root     17372  0.1  0.0 153892  7844 ?        Ssl  8月15   3:21 /usr/local/bin/redis-server *:6380 [cluster]
root     17377  0.1  0.0 153892  7848 ?        Ssl  8月15   3:20 /usr/local/bin/redis-server *:6381 [cluster]
[root@host-192-125-30-111 ~]# kill -9 17367
[root@host-192-125-30-111 ~]# kill -9 17372
[root@host-192-125-30-111 ~]# kill -9 17377

# 删除数据目录
[root@host-192-125-30-111 ~]# cd /usr/local/
[root@host-192-125-30-111 local]# ls
bin  etc  games  include  lib  lib64  libexec  redis-cluster  sbin  share  src
[root@host-192-125-30-111 local]# cd redis-cluster
[root@host-192-125-30-111 redis-cluster]# ls
data  log  redis-6379.conf  redis-6380.conf  redis-6381.conf  redis.conf
[root@host-192-125-30-111 redis-cluster]# cd data
[root@host-192-125-30-111 data]# ls
6379  6380  6381
[root@host-192-125-30-111 data]# rm -rf 63*
[root@host-192-125-30-111 data]# ls
[root@host-192-125-30-111 data]# mkdir -p 6379 6380 6381
[root@host-192-125-30-111 data]# cd ..
[root@host-192-125-30-111 redis-cluster]# ls
data  log  redis-6379.conf  redis-6380.conf  redis-6381.conf  redis.conf
[root@host-192-125-30-111 redis-cluster]# cd log
[root@host-192-125-30-111 log]# ls
redis-6379.log  redis-6380.log  redis-6381.log
[root@host-192-125-30-111 log]# rm -rf redis-*.log
[root@host-192-125-30-111 log]# ls
[root@host-192-125-30-111 log]# 
```

`redis-cli` 的参数说明：

```sh
# 表示创建一个redis集群。
create

# 表示为集群中的每一个主节点指定一个从节点，即一比一的复制。
--cluster-replicas 1
```

注意：如果配置项 `cluster-enabled` 的值不为yes，则执行时会报错 `[ERR] Node 192.168.0.251:6381 is not configured as a cluster node.`

输出：

```sh
[root@host-192-125-30-111 redis-cluster]# /usr/local/bin/redis-cli -a Perfect1 --cluster create 192.125.30.111:6379 192.125.30.111:6380 192.125.30.59:6381 192.125.30.59:6379 192.125.30.59:6380 192.125.30.111:6381 --cluster-replicas 1
Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.125.30.59:6380 to 192.125.30.111:6379
Adding replica 192.125.30.111:6381 to 192.125.30.59:6381
Adding replica 192.125.30.59:6379 to 192.125.30.111:6380
M: 9df266851fa7d8d9363d584a5bc644f36ea1d052 192.125.30.111:6379
   slots:[0-5460] (5461 slots) master
M: 932a6ab157056e2d36e3915780b5d77138bbabdb 192.125.30.111:6380
   slots:[10923-16383] (5461 slots) master
M: fb9923083a70ca2131b3fb6c7633bc7275dfe33c 192.125.30.59:6381
   slots:[5461-10922] (5462 slots) master
S: 53ab782834f304013ac055a35e5275e5004b30c7 192.125.30.59:6379
   replicates 932a6ab157056e2d36e3915780b5d77138bbabdb
S: 7854f8ed0234945e89af5079331ce7779449d632 192.125.30.59:6380
   replicates 9df266851fa7d8d9363d584a5bc644f36ea1d052
S: 1141e28e7b135db55cdb1c87c143ee0c0502d26d 192.125.30.111:6381
   replicates fb9923083a70ca2131b3fb6c7633bc7275dfe33c
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
..
>>> Performing Cluster Check (using node 192.125.30.111:6379)
M: 9df266851fa7d8d9363d584a5bc644f36ea1d052 192.125.30.111:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 1141e28e7b135db55cdb1c87c143ee0c0502d26d 192.125.30.111:6381
   slots: (0 slots) slave
   replicates fb9923083a70ca2131b3fb6c7633bc7275dfe33c
M: 932a6ab157056e2d36e3915780b5d77138bbabdb 192.125.30.111:6380
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 7854f8ed0234945e89af5079331ce7779449d632 192.125.30.59:6380
   slots: (0 slots) slave
   replicates 9df266851fa7d8d9363d584a5bc644f36ea1d052
S: 53ab782834f304013ac055a35e5275e5004b30c7 192.125.30.59:6379
   slots: (0 slots) slave
   replicates 932a6ab157056e2d36e3915780b5d77138bbabdb
M: fb9923083a70ca2131b3fb6c7633bc7275dfe33c 192.125.30.59:6381
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

**集群相关信息查看**

```sh
# 集群状态
/usr/local/bin/redis-cli -h 127.0.0.1 -p 6379 -a password cluster info

# 集群节点信息
/usr/local/bin/redis-cli -h 127.0.0.1 -p 6379 -a password cluster nodes

# 节点内存、cpu、key数量等信息（每个节点都需查看）
/usr/local/bin/redis-cli -h 127.0.0.1 -p 6379 -a password info
```

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

```sh
$ ./redis-cli --cluster del-node 192.168.0.251:6381 082c079149a9915612d21cca8e08c831a4edeade

>>> Removing node 082c079149a9915612d21cca8e08c831a4edeade from cluster 192.168.0.251:6381

>>> Sending CLUSTER FORGET messages to the cluster...

>>> SHUTDOWN the node.
```

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

[Redis集群](https://www.cnblogs.com/cjsblog/p/9048545.html)

[分布式缓存 Redis 集群搭建](https://www.cnblogs.com/esofar/p/10486621.html)

[redis集群配置文件](https://www.cnblogs.com/zh-dream/p/12275031.html)

[Redis详解（二）-- redis的配置文件介绍](https://www.cnblogs.com/ysocean/p/9074787.html)

[高可用Redis(十一)：使用redis-trib.rb工具搭建集群](https://www.cnblogs.com/renpingsheng/p/9833740.html)

[Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)

[Redis Cluster Specification](https://redis.io/topics/cluster-spec)