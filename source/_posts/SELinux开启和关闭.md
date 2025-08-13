---
title: SELinux开启和关闭
date: 2020-08-21 11:14:40
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 查看SELinux状态

```sh
# 1, 如果SELinux status参数为enabled即为开启状态
/usr/sbin/sestatus -v
SELinux status:                 enabled

# 2,也可以用这个命令检查
getenforce
```

## 关闭SELinux方法

**临时关闭（不用重启机器）**

```sh
# 设置SELinux 成为permissive模式
setenforce 0

# 设置SELinux 成为enforcing模式
#setenforce 1
```

**修改配置文件（需要重启机器）**

修改 `/etc/selinux/config` 文件

将 `SELINUX=enforcing` 改为 `SELINUX=disabled`

```sh
vim /etc/selinux/config
```

参考

[Linux下查看SELinux状态和关闭SELinux的方法](https://www.jb51.net/LINUXjishu/192576.html)