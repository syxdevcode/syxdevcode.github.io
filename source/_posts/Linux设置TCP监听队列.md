---
title: Linux设置TCP监听队列
date: 2020-08-04 17:42:10
tags:
- Linux
- CentOS7
categories:
- Linux
---

tcp的三次握手:

![微信截图_20200804174353.png](/img/微信截图_20200804174353.png)

1、client发送SYN到server，将状态修改为SYN_SEND，如果server收到请求，则将状态修改为SYN_RCVD，并把该请求放到syns queue队列中。
2、server回复SYN+ACK给client，如果client收到请求，则将状态修改为ESTABLISHED，并发送ACK给server。
3、server收到ACK，将状态修改为ESTABLISHED，并把该请求从syns queue中放到accept queue。

在linux系统内核中维护了两个队列：`syns queue` 和 `accept queue`

**syns queue**

用于保存半连接状态的请求，其大小通过 `/proc/sys/net/ipv4/tcp_max_syn_backlog` 指定，不过这个设置有效的前提是系统的syncookies功能被禁用。互联网常见的 `TCP SYN FLOOD`(拒绝服务攻击) 恶意DOS攻击方式就是建立大量的半连接状态的请求，然后丢弃，导致 `syns queue` 不能保存其它正常的请求。

**accept queue**

用于保存全连接状态的请求，即 `TCP listen` 的 `backlog` 大小，通过 `/proc/sys/net/core/somaxconn` 指，默认值一般较小如128，需要修改大一点，比如改成32767。

如果 `accpet queue` 队列满了，server将发送一个ECONNREFUSED错误信息Connection refused到client。

立即生效还可以使用命令：`sysctl -w net.core.somaxconn=32767`。

要想永久生效，需要在文件 `/etc/sysctl.conf` 中增加一行：

`net.core.somaxconn = 32767`，

或者使用命令：

`echo "net.core.somaxconn = 32767" >> /etc/sysctl.conf`,

然后执行命令 `sysctl -p` 以生效。

Redis配置项 `tcp-backlog` 的值不能超过somaxconn的大小。

参考：

[浅谈tcp socket的backlog参数](https://www.jianshu.com/p/e6f2036621f4)

[Linux TCP队列相关参数的总结](https://developer.aliyun.com/article/4252)