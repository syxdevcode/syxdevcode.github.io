---
title: 查看Linux系统版本
date: 2021-01-29 13:57:29
tags:
- Linux
- CentOS7
- Linux优化
categories:
- Linux
---

## 查看Linux内核版本

```sh
cat /proc/version
uname -a
```

## 查看Linux系统版本

```sh
# 这个命令适用于所有的Linux发行版，包括RedHat、SUSE、Debian…等发行版。
# 如果未安装，需要命令：yum install redhat-lsb -y
lsb_release -a

# 只适合Redhat系的Linux
cat /etc/redhat-release
file /bin/bash

# 适用于所有的Linux发行版
cat /etc/issue
```
