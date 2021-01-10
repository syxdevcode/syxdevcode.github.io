---
title: Linux TcpDump工具
date: 2021-01-08 17:40:49
tags:
- Linux
- CentOS7
- 网络抓包
- TcpDump
- Linux优化
categories:
- TcpDump
---

## 安装

```sh
# 解锁
# 如果不解锁，可能造成错误：Couldn't find user 'tcpdump'
chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow

# 安装
yum install  tcpdump
```

## 使用


















参考：

[tcpdump抓包wireshark分析](https://blog.csdn.net/godop/article/details/80986966)

[Linux基础：用tcpdump抓包](https://www.cnblogs.com/chyingp/p/linux-command-tcpdump.html)

[聊聊 tcpdump 与 Wireshark 抓包分析](https://www.jianshu.com/p/8d9accf1d2f1)