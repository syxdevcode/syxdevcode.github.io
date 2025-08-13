---
title: >-
  ERROR 2002 (HY000): Can't connect to local MySQL server through socket
  '/tmp/mysql.sock' (2)
date: 2020-09-09 14:12:55
tags:
- Linux
- MySql
- CentOS7
- 安装部署
categories: MySql
---

这个错是链接时报的错，要链接必须启动。修复的时候首先要启动mysql。

首先来了解一下 `mysql.sock` 的作用：

Mysql有两种连接方式：

（1）TCP/IP
（2）socket

对`mysql.sock`来说，其作用是程序与 `mysql server` 处于同一台机器，发起本地连接时可用。
例如你无须定义连接 `host` 的具体IP，只要为空或 `localhost` 就可以。
在此种情况下，即使你改变 `mysql` 的外部 `port` 也是一样可能正常连接。
因为你在 `my.ini` 中或 `my.cnf` 中改变端口后，`mysql.sock` 是随每一次 `mysql server`启动生成的。已经根据你在更改完 `my.cnf` 后重启 `mysql` 时重新生成了一次，信息已跟着变更。

那么对于外部连接，必须是要变更port才能连接的。
`mysql.sock` 的作用是 `server` 和 `client` 在同一台服务器，并且使用 `localhost` 进行链接的时候，就会使用socket来进行连接——仅此而已
也就是：为主机名为 `localhost` 建立的MySQL连接，该连接过程通过一个套接字文件 `mysql.socket` 实现的。所以该文件被删后，用 `localhost` 用户是连接不到MySQL服务器的。

必须建立一条 `tcp/ip` 连接，即使用 `127.0.0.1` 而不是 `localhost` 作为-h的参数去连接MySQL服务器，如：`mysqladmin -h 127.0.0.1 -u root -p shutdown`，强制地建立一条 `tcp/ip` 连接；
关闭MySQL服务器，再重新以 `localhost` 为主机名启动MySQL服务器，它就会重新创建一个套接字文件。

注意：`mysql.sock` 是一个临时文件，启动mysql后才会生成。
报错的原因：MySQL将其放在 `/tmp` 目录，而linux将其放在 `/var/mysql` 目录。所以我们只需要创建一个软链接，输入以下两个命令即可：
如果 `/var/` 下没有 `mysql`目录，则需创建

创建目录：`sudo mkdir /var/mysql`

创建软链接：`sudo ln -s /var/mysql/mysql.sock /tmp/mysql.sock`

如果提示： `ln: creating symbolic link `/var/lib/mysql/mysql.sock': File exists`
删除之前的 `mysql.sock` 文件。

参考：

[ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)](https://www.cnblogs.com/Lam7/p/6090975.html)

[MySql 报错ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)](https://blog.csdn.net/github_37216944/article/details/78843685)