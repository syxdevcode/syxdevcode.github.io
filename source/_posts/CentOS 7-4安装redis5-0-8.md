---
title: CentOS7.4安装redis5.0.8
date: 2020-05-12 16:01:42
tags:
- Linux
- CentOS7
- Redis
- 安装部署
categories:
- Redis
---

## 下载

[官网](https://redis.io/ "官网")下载

下载地址：[redis-5.0.8.tar.gz](http://download.redis.io/releases/redis-5.0.8.tar.gz)

## 安装

### 安装gcc依赖

````linux
yum install gcc
````

### 解压安装

````linux
tar -zxvf redis-5.0.8.tar.gz
cd redis-5.0.8
cd src
make distclean # make distclean清除之前生成的文件（可选）
make && make install
````

### 修改配置文件

编辑 `redis.conf`:

```sh
vim /ldjc/rj/redis-5.0.8/redis.conf
```

进入vim 输入 `/` 再按 回车用来搜索关键字

比如如下命令会定位到文件中出现 bind 的位置并会将关键词用颜色区分。

`/bind`

* 1，后台启动

搜索 `daemonize no` 改为 `daemonize yes`

* 2，设置密码

搜索 `requirepass password` 添加 `requirepass 设置的密码`

`requirepass password`

* 3，外部访问

搜索 `bind 127.0.0.1` 然后使用 `#`注释掉

```sh
# bind 127.0.0.1
```

### 启动

命令:

```sh
cd /ldjc/rj/redis-5.0.8

# 启动
/usr/local/bin/redis-server /ldjc/rj/redis-5.0.8/redis.conf

# 连接
/usr/local/bin/redis-cli
```

示例：

```cs
[root@host-192-125-30-11 redis-5.0.8]# /usr/local/bin/redis-server /ldjc/rj/redis-5.0.8/redis.conf
15531:C 12 May 2020 16:07:35.523 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
15531:C 12 May 2020 16:07:35.523 # Redis version=5.0.8, bits=64, commit=00000000, modified=0, pid=15531, just started
15531:C 12 May 2020 16:07:35.523 # Configuration loaded
[root@host-192-125-30-11 redis-5.0.8]# /usr/local/bin/redis-cli
127.0.0.1:6379> keys *
(error) NOAUTH Authentication required.
127.0.0.1:6379> auth root
OK
127.0.0.1:6379> keys *
(empty list or set)
127.0.0.1:6379> 
```

### 开放端口

注意：针对各种云，因为有专门策略，所以不需要操作防火墙；

* 开放6379端口

```sh
firewall-cmd --zone=public --add-port=6379/tcp --permanent

#重启防火墙
firewall-cmd --reload

## 重新查询端口是否开放
firewall-cmd --query-port=6379/tcp

## 查看监听的端口
netstat -lnpt

#关闭6379端口
firewall-cmd --zone=public --remove-port=6379/tcp --permanent  

## 查看防火墙所有开放的端口
firewall-cmd --zone=public --list-ports
```

* 查找redis
ps aux | grep redis

### 关闭Redis服务

```linux
[root@localhost bin]# pkill redis-server
[root@localhost bin]# netstat -tunpl | grep 6379
[root@localhost bin]# pstree -p | grep redis
[root@localhost bin]# /usr/local/bin/redis-cli
Could not connect to Redis at 127.0.0.1:6379: Connection refused
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected>
```