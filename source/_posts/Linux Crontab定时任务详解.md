---
title: Linux Crontab定时任务详解
date: 2020-08-06 17:38:28
tags:
- Linux
- CentOS7
- 定时任务
- Crontab
categories:
- Crontab
---

## Linux crontab简介

Linux crontab是用来定期执行程序的命令。
当安装完成操作系统之后，默认便会启动此任务调度命令。

注意：新创建的 cron 任务，不会马上执行，至少要过 2 分钟后才可以，当然你可以重启 cron 来马上执行。

crontab命令用于设置周期性被执行的指令。该命令从标准输入设备读取指令，并将其存放于 `/etc/crontab` 文件中，以供之后读取和执行。

cron 系统调度进程。 可以使用它在每天的非高峰负荷时间段运行作业，或在一周或一月中的不同时段运行。cron是系统主要的调度进程，可以在无需人工干预的情况下运行作业。crontab命令允许用户提交、编辑或删除相应的作业。每一个用户都可以有一个crontab文件来保存调度信息。

linux 任务调度的工作主要分为以下两类：

1、系统执行的工作：系统周期性所要执行的工作，如备份系统数据、清理缓存
2、个人执行的工作：某个用户定期要做的工作，例如每隔10分钟检查邮件服务器是否有新信，这些工作可由每个用户自行设置

## crontab 命令

crontab命令是 `cron table` 的简写，它是 cron 的配置文件，也可以叫它作业列表，我们可以在以下文件夹内找到相关配置文件。

`/var/spool/cron/` 目录下存放的是每个用户包括root的crontab任务，每个任务以创建者的名字命名
`/etc/crontab` 这个文件负责调度各种管理和维护任务。
`/etc/cron.d/` 这个目录用来存放任何要执行的crontab文件或脚本。

**`/etc/crontab`**

```sh
more /etc/crontab
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

**`/var/spool/cron`**

每个用户都会生成一个自动生成一个自己的 `crontab` 文件，一般位于 `/var/spool/cron` 目录下

```sh
cd /var/spool/cron
```

如果你用命令 `crontab -r` 就会删除当前用户的crontab文件，例如你切换到oracle账号下，执行了该命令，那么 `/var/spool/cron/oracle` 文件就会删除，如果要创建该文件只需要用 `crontab -e` 命令即可。注意，普通用户一般没有权限访问 `/var/spool/cron`

我们还可以把脚本放在 `/etc/cron.hourly`、`/etc/cron.daily`、`/etc/cron.weekly`、`/etc/cron.monthly` 目录中，让它每小时/天/星期、月执行一次。

系统管理员可以通过 cron.deny 和 cron.allow 这两个文件来禁止或允许用户拥有自己的crontab文件。

`/etc/cron.deny` 表示不能使用 `crontab` 命令的用户

`/etc/cron.allow` 表示能使用 `crontab` 的用户。

默认情况下，`cron.allow` 文件不存在。如果两个文件同时存在，那么 `/etc/cron.allow` 优先。如果两个文件都不存在，那么只有超级用户可以安排作业。

**`ls -lrt cron*`**

```sh
ls -lrt cron*
-rw-r--r--. 1 root root 451 6月  10 2014 crontab
-rw-------. 1 root root   0 11月 20 2018 cron.deny

cron.weekly:
总用量 0

cron.monthly:
总用量 0

cron.hourly:
总用量 4
-rwxr-xr-x. 1 root root 392 11月 20 2018 0anacron

cron.daily:
总用量 8
-rwxr-xr-x. 1 root root 618 10月 30 2018 man-db.cron
-rwx------. 1 root root 219 10月 31 2018 logrotate

cron.d:
总用量 4
-rw-r--r--. 1 root root 128 11月 20 2018 0hourl
```

## CRONTAB在线手册

注意：不同版本的Linux系统，可能crontab手册内容有所出入，请以实际版本为准。

```sh
man crontab | more
```

## 注意事项

配置定时任务时，需要注意两个问题：

1: 在SHELL中设置了必要的环境变量；例如一个shell脚本手工执行OK，但是配置成后台作业执行时，获取不到ORACLE的环境变量，这是因为crontab环境变量问题，Crontab的环境默认情况下并不包含系统中当前用户的环境。所以，你需要在shell脚本中添加必要的环境变量的设置

2: 尽量所有的文件都采用完全路径方式，避免使用相对路径。

参考：

[Linux crontab定时任务配置方法(详解)](https://www.jb51.net/article/98640.htm)

[Linux Crontab 定时任务](https://www.runoob.com/w3cnote/linux-crontab-tasks.html)

[Linux crontab 命令](https://www.runoob.com/linux/linux-comm-crontab.html)

[CentOS安装配置Crontab定时任务](https://www.linuxidc.com/Linux/2019-03/157851.htm)