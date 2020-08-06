---
title: 使用Linux自带日志滚动工具logrotate滚动redis日志示例
date: 2020-08-06 15:51:36
tags:
- Linux
- CentOS7
- Redis
- 安装部署
- 日志
categories:
- Redis
---

转载：[使用Linux自带日志滚动工具logrotate滚动redis日志示例](http://blog.chinaunix.net/uid-20682147-id-5818053.html)

截至到redis-5.0版本，redis仍然不会自动滚动日志文件，如果不处理则日志文件日积月累越来越大，最终将导致磁盘满告警，

使用命令查看磁盘使用情况：

```sh
ls -lh
```

Linux系统自带的日志滚动工具logrotate由两部分组成：

一是命令行工具logrotate;
二是后台服务rsyslogd。

使用 rsyslogd，只需简单的配置即可实现日志滚动。
rsyslogd 的配置文件为 `/etc/logrotate.conf`，但一般不建议直接修改 `logrotate.conf` ，而是在目录 `/etc/logrotate.d` 下新增文件的方式。
`logrotate.conf` 会include所有logrotate.d目录下的文件，
语法是一致的，区别是 `logrotate.conf` 定义了默认的配置，而 `logrotate.d` 目录下为专有配置。

下列为redis的配置示例：

```sh
# cat /etc/logrotate.d/redis
# 以下为 redis 文件的内容：
/usr/local/redis-cluster/log/redis-6379.log
/usr/local/redis-cluster/log/redis-6380.log
/usr/local/redis-cluster/log/redis-6381.log
{
    rotate 2
    minsize 100M
    nocompress
    missingok
    create 0664 redis redis
    notifempty
}
```
 

配置项说明：

1) rotate指定日志文件备份数，如果值为0表示不备份
2) minsize表示日志文件达到多大才滚动
3) nocompress表示是否压缩备份的日志文件
4) missingok如果日志丢失，不报错继续滚动下一个日志
5) notifempty日志文件为空时，不进行轮转，默认值为ifempty
6) create指定创建新日志文件的属性，logrotate是以root运行的，如果目标日志文件非root运行，则这个一定要指定好。

注意，修改后需要重启下rsyslogd。如果是CentOS可使用下列任意一种方式重启
（实际上systemctl新方式，而service实际也是使用systemctl）：

```sh
# service rsyslog restart 
# systemctl restart  rsyslog.service
```