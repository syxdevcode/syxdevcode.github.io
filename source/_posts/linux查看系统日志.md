---
title: linux查看系统日志
date: 2020-09-28 10:19:47
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 常见文件注解

linux系统文件通常在 `/var/log`

```sh
# 系统启动后的信息和错误日志
/var/log/message

# 与安全相关的日志信息
/var/log/secure

# 与邮件相关的日志信息
/var/log/maillog

# 与定时任务相关的日志信息
/var/log/cron

# 与UUCP和news设备相关的日志信息
/var/log/spooler

# 守护进程启动和停止相关的日志消息
/var/log/boot.log

# 永久记录每个用户登录、注销及系统的启动、停机的事件
/var/log/wtmp

# 记录当前正在登录系统的用户信息
/var/run/utmp

# 记录失败的登录尝试信息
/var/log/btmp
```

## 历史记录

```sh
# 查看重启的命令
last | grep reboot

# 历史操作
history

# 立即清空当前所有的历史记录
history -c 
```

## 最近重启记录

```sh
# 查看所有重启日志信息
last reboot

# 查看最近的一条
last reboot | head -1

last -x|grep shutdown | head -1

# -x：显示系统关机和运行等级改变信息
last
last -x
last -x reboot
last -x shutdown

# 开机时间
uptime -s
```

参考：

[linux系统重启 查看日志及历史记录](https://blog.csdn.net/gzy19870815/article/details/88632729)