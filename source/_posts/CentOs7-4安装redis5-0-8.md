---
title: CentOs7.4安装redis5.0.8
date: 2020-05-12 16:01:42
tags:
---

8.1 修改配置文件
vim /usr/local/redis/redis-5.0.0/redis.conf

进入vim 输入 / 再按火车用来搜索关键字

比如如下命令会定位到文件中出现 bind 的位置并会将关键词用颜色区分。

/bind

 8.2 后台启动
搜索 daemonize no 改为 daemonize yes

daemonize no

 8.3 设置密码
搜索 requirepass foobared 添加 requirepass 你设置的密码

requirepass foobared

 8.4 外部访问
搜索 bind 127.0.0.1 然后注释掉

# bind 127.0.0.1

记得开放6379端口

firewall-cmd --zone=public --add-port=6379/tcp --permanent
firewall-cmd --reload



ps aux | grep redis




cd /ldjc/rj/redis-5.0.8

src/redis-server redis.conf

src/redis-cli 

```sh
[root@host-192-125-30-111 redis-5.0.8]# src/redis-server redis.conf
15531:C 12 May 2020 16:07:35.523 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
15531:C 12 May 2020 16:07:35.523 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=15531, just started
15531:C 12 May 2020 16:07:35.523 # Configuration loaded
[root@host-192-125-30-111 redis-5.0.8]# src/redis-cli
127.0.0.1:6379> keys *
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth Perfect1
OK
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> 
```