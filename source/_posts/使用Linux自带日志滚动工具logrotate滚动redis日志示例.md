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

## Linux logrotate简介

`Linux logrotate` 命令用于管理记录文件。

使用 `logrotate` 指令，可让你轻松管理系统所产生的记录文件。它提供自动替换，压缩，删除和邮寄记录文件，每个记录文件都可被设置成每日，每周或每月处理，也能在文件太大时立即处理。例如，你可以设置 `logrotate`，让 `/var/log/foo` 日志文件每30天轮循，并删除超过6个月的日志。配置完后，`logrotate` 的运作完全自动化，不必进行任何进一步的人为干预。

`logrotate` 需要的 `cron` 任务应该在安装时就自动创建好了，下面代码以centos7.4为例：

```sh
cat /etc/cron.daily/logrotate
#!/bin/sh

/usr/sbin/logrotate -s /var/lib/logrotate/logrotate.status /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0

```

Linux系统自带的日志滚动工具 `logrotate` 由两部分组成：

一是命令行工具 `logrotate`;
二是后台服务 `rsyslogd`。

使用 `rsyslogd` ，只需简单的配置即可实现日志滚动。
`rsyslogd` 的配置文件为 `/etc/logrotate.conf`，但一般不建议直接修改 `logrotate.conf` ，而是在目录 `/etc/logrotate.d` 下新增文件的方式。
`logrotate.conf` 会 `include` 所有 `logrotate.d` 目录下的文件，
语法是一致的，区别是 `logrotate.conf` 定义了默认的配置，而 `logrotate.d` 目录下为专有配置。

## Redis日志设置

截至到 `redis-5.0` 版本，redis 仍然不会自动滚动日志文件，如果不处理则日志文件日积月累越来越大，最终将导致磁盘满告警，

使用命令查看磁盘使用情况：

```sh
ls -lh
```

下列为redis的配置示例：

```sh
# cat /etc/logrotate.d/redis
# 以下为 redis 文件的内容：
/usr/local/redis-cluster/log/redis-6379.log
/usr/local/redis-cluster/log/redis-6380.log
/usr/local/redis-cluster/log/redis-6381.log
{
    monthly
    size=150M
    rotate 5
    minsize 100M
    nocompress
    missingok
    notifempty
    create 0664 root root
    postrotate
        /usr/bin/killall -HUP rsyslogd
    endscript
}
```

注：`postrotate/endscript` 可以去掉，即：

```sh
# cat /etc/logrotate.d/redis
# 以下为 redis 文件的内容：
/usr/local/redis-cluster/log/redis-6379.log
/usr/local/redis-cluster/log/redis-6380.log
/usr/local/redis-cluster/log/redis-6381.log
{
    monthly
    size=150M
    rotate 5
    minsize 100M
    nocompress
    missingok
    notifempty
    create 0664 root root
}
```

配置项说明：

可以使用 `man logrotate` 命令查看详情。

* `monthly`: 日志文件将按月轮循。其它可用值为 `daily`，`weekly` 或者 `yearly`。
* `rotate` 指定日志文件备份数，一次将存储5个归档日志。对于第六个归档，时间最久的归档将被删除，如果值为0表示不备份
* `minsize` 表示日志文件达到多大才滚动
* `nocompress` 表示是否压缩备份的日志文件，取值：`compress`，`nocompress`
* `delaycompress`: 总是与 `compress` 选项一起用，`delaycompress` 选项指示 `logrotate` 不要将最近的归档压缩，压缩将在下一次轮循周期进行。这在你或任何软件仍然需要读取最新归档时很有用。
* `missingok` 在日志轮循期间，任何错误将被忽略
* `notifempty` 日志文件为空时，不进行轮转，默认值为 `ifempty`
* `create` 以指定的权限创建全新的日志文件，同时logrotate也会重命名原始日志文件。`logrotate` 是以 root 运行的，如果目标日志文件非root运行，则这个一定要指定好。
* `postrotate/endscript`: 在所有其它指令完成后，`postrotate` 和 `endscript` 里面指定的命令将被执行。在这种情况下，`rsyslogd` 进程将立即再次读取其配置并继续运行。
* size : size 当日志文件到达指定的大小时才转储，Size 可以指定 bytes (缺省)以及KB (sizek)或者MB (sizem)。

注意，修改后需要重启下 `rsyslogd`。如果是 `CentOS` 可使用下列任意一种方式重启
（实际上 `systemctl` 新方式，而 `service` 实际也是使用 `systemctl` ）：

```sh
service rsyslog restart 
systemctl restart rsyslog.service
```

## 实例

**1，只轮循一个日志文件，日志文件大小可以增长到50MB**

```sh
vim /etc/logrotate.d/log-file

# 以下为内容
/var/log/log-file {
    size=50M
    rotate 5
    create 644 root root
    postrotate
        /usr/bin/killall -HUP rsyslogd
    endscript
}
```

**日志文件以创建日期命名**

使用 `dateext` 实现。

```sh
vim /etc/logrotate.d/log-file

# 以下为内容
/var/log/log-file {
    monthly
    rotate 5
    dateext
    create 644 root root
    postrotate
        /usr/bin/killall -HUP rsyslogd
    endscript
}
```

## 手动运行

logrotate可以在任何时候从命令行手动调用。

要调用为 `/etc/lograte.d/` 下配置的所有日志调用 `logrotate` ：

`logrotate /etc/logrotate.conf` 

要为某个特定的配置调用 `logrotate` ：

`logrotate /etc/logrotate.d/log-file` 

**logrotate参数说明**

```sh
# -d 选项以预演方式运行logrotate
logrotate -d /etc/logrotate.d/log-file 

# -f 强制logrotate轮循日志文件
# -v 参数提供了详细的输出。
logrotate -vf /etc/logrotate.d/log-file

# 输出到指定的文件
# 注：logrotate自身的日志通常存放于/var/lib/logrotate/status目录。
logrotate -vf –s /var/log/logrotate-status /etc/logrotate.d/log-file
```

参考：

[使用Linux自带日志滚动工具logrotate滚动redis日志示例](http://blog.chinaunix.net/uid-20682147-id-5818053.html)

[Linux日志文件总管——logrotate](https://linux.cn/article-4126-1.html)

[利用Linux自带的logrotate管理日志](https://www.cnblogs.com/miaocbin/p/11540312.html)

[Linux logrotate命令](https://www.runoob.com/linux/linux-comm-logrotate.html)