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


## 配置redis

指定端口的配置文件 `redis-PORT.conf`；
该文件定义所有与端口相关的配置项，PORT需要替换为具体的端口，如6379

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
dir /data/redis/data/PORT

# 纯文件名，位于dir指定的目录下，不能包含目录，
# 否则报错“appendfilename can't be a path, just # a filename”。
# 如果开启了AOF，REdis进程启动时并不会读取RDB文件，
# 所以配置上可以考虑关闭RDB，这样可以提升REdis稳定性。
dbfilename dump-PORT.rdb

# 纯文件名，位于dir指定的目录下，不能包含目录，
# 否则报错“appendfilename can't be a path, just a filename”
appendfilename "appendonly-PORT.aof"

# 日志文件，包含目录和文件名，注意redis不会自动滚动日志文件
logfile /data/redis/log/redis-PORT.log

# yes表示以集群方式运行，为no表示以非集群方式运行
cluster-enabled yes

# 日志级别，建议为notice，
# 另外注意redis是不会滚动日志文件的，每次写日志都是先打开日志文件再写日志再关闭方式
# debug
# verbose
# notice
# warning
loglevel notice

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

# 设置最大的内存，单位为字节 26843545600(25GB)
# 注意，在 64bit 系统下，maxmemory 设置为 0 表示不限制 Redis 内存使用，
# 在 32bit 系统下，maxmemory 隐式不能超过 3GB。 
maxmemory 26843545600

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













参考：

[分布式缓存 Redis 集群搭建](https://www.cnblogs.com/esofar/p/10486621.html)

[redis集群配置文件](https://www.cnblogs.com/zh-dream/p/12275031.html)

[Redis详解（二）-- redis的配置文件介绍](https://www.cnblogs.com/ysocean/p/9074787.html)

[Redis-5.0.0集群配置](https://blog.csdn.net/Aquester/article/details/90511200)

[Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)

[Redis Cluster Specification](https://redis.io/topics/cluster-spec)