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

crond 命令每分锺会定期检查是否有要执行的工作，如果有要执行的工作便会自动执行该工作。

注意：新创建的 cron 任务，不会马上执行，至少要过 2 分钟后才可以，当然你可以重启 cron 来马上执行。

而 linux 任务调度的工作主要分为以下两类：

1、系统执行的工作：系统周期性所要执行的工作，如备份系统数据、清理缓存
2、个人执行的工作：某个用户定期要做的工作，例如每隔10分钟检查邮件服务器是否有新信，这些工作可由每个用户自行设置

## crontab命令

crontab命令是 `cron table` 的简写，它是 cron的配置文件，也可以叫它作业列表，我们可以在以下文件夹内找到相关配置文件。

`/var/spool/cron/` 目录下存放的是每个用户包括root的crontab任务，每个任务以创建者的名字命名
`/etc/crontab` 这个文件负责调度各种管理和维护任务。
`/etc/cron.d/` 这个目录用来存放任何要执行的crontab文件或脚本。

我们还可以把脚本放在 `/etc/cron.hourly`、`/etc/cron.daily`、`/etc/cron.weekly`、`/etc/cron.monthly` 目录中，让它每小时/天/星期、月执行一次。


crontab命令用于设置周期性被执行的指令。该命令从标准输入设备读取指令，并将其存放于 `/etc/crontab` 文件中，以供之后读取和执行。

cron 系统调度进程。 可以使用它在每天的非高峰负荷时间段运行作业，或在一周或一月中的不同时段运行。cron是系统主要的调度进程，可以在无需人工干预的情况下运行作业。crontab命令允许用户提交、编辑或删除相应的作业。每一个用户都可以有一个crontab文件来保存调度信息。系统管理员可以通过cron.deny 和 cron.allow 这两个文件来禁止或允许用户拥有自己的crontab文件。



参考：

[Linux crontab定时任务配置方法(详解)](https://www.jb51.net/article/98640.htm)

[Linux Crontab 定时任务](https://www.runoob.com/w3cnote/linux-crontab-tasks.html)

[Linux crontab 命令](https://www.runoob.com/linux/linux-comm-crontab.html)

[CentOS安装配置Crontab定时任务](https://www.linuxidc.com/Linux/2019-03/157851.htm)