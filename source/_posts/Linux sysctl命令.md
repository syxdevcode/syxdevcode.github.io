---
title: Linux sysctl命令
date: 2020-08-04 16:43:11
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 简介

sysctl命令被用于在内核运行时动态地修改内核的运行参数，可用的内核参数在目录/proc/sys中。它包含一些TCP/ip堆栈和虚拟内存系统的高级选项， 这可以让有经验的管理员提高引人注目的系统性能。用sysctl可以读取设置超过五百个系统变量。

参数：

```sh
-n：打印值时不打印关键字；
-e：忽略未知关键字错误；
-N：仅打印名称；
-w：当改变sysctl设置时使用此项；
-p：从配置文件“/etc/sysctl.conf”加载内核参数设置；
-a：打印当前所有可用的内核参数变量和值；
-A：以表格方式打印当前所有可用的内核参数变量和值。
```

## 实例

查看所有可读变量：

`sysctl -a`

```sh
[root@host-192-125-30-59 ~]# sysctl -a | grep file-max
fs.file-max = 6523120
sysctl: reading key "net.ipv6.conf.all.stable_secret"
sysctl: reading key "net.ipv6.conf.default.stable_secret"
sysctl: reading key "net.ipv6.conf.eth0.stable_secret"
sysctl: reading key "net.ipv6.conf.lo.stable_secret"
```

参考：

[sysctl命令](https://man.linuxde.net/sysctl)