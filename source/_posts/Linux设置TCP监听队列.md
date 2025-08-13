---
title: Linux设置TCP监听队列
date: 2020-08-04 17:42:10
tags:
- Linux
- CentOS7
- Linux优化
- TCP协议
categories:
- TCP协议
---

tcp的三次握手:

![微信截图_20200804174353.png](/img/微信截图_20200804174353.png)

1、client发送SYN到server，将状态修改为SYN_SEND，如果server收到请求，则将状态修改为SYN_RCVD，并把该请求放到syns queue队列中。
2、server回复SYN+ACK给client，如果client收到请求，则将状态修改为ESTABLISHED，并发送ACK给server。
3、server收到ACK，将状态修改为ESTABLISHED，并把该请求从 `syns queue` 中放到 `accept queue`。

## backlog参数

backlog其实是一个连接队列，在Linux内核2.2之前，backlog大小包括半连接状态和全连接状态两种队列大小。

* 半连接状态为：服务器处于Listen状态时收到客户端SYN报文时放入半连接队列中，即 `SYN queue`（服务器端口状态为：SYN_RCVD）。
* 全连接状态为：TCP的连接状态从服务器（SYN+ACK）响应客户端后，到客户端的ACK报文到达服务器之前，则一直保留在半连接状态中；当服务器接收到客户端的ACK报文后，该条目将从半连接队列搬到全连接队列尾部，即 `accept queue` （服务器端口状态为：ESTABLISHED）。

在Linux内核2.2之后，分离为两个backlog来分别限制半连接（SYN_RCVD状态）队列大小和全连接（ESTABLISHED状态）队列大小。

![927655-20161215133843776-605308204.png](/img/927655-20161215133843776-605308204.png)

**syns queue**

SYN queue 用于保存半连接状态的请求，队列长度由 `/proc/sys/net/ipv4/tcp_max_syn_backlog` 指定，默认为1024。

需要在文件 `/etc/sysctl.conf` 中增加一行：

`net.ipv4.tcp_max_syn_backlog = 262144`

互联网常见的 `TCP SYN FLOOD`(拒绝服务攻击) 恶意DOS攻击方式就是建立大量的半连接状态的请求，然后丢弃，导致 `syns queue` 不能保存其它正常的请求。

**accept queue**

Accept queue 用于保存全连接状态的请求，队列长度由 `/proc/sys/net/core/somaxconn` 和使用 `listen` 函数时传入的参数，二者取最小值。默认为128。在Linux内核2.4.25之前，是写死在代码常量 `SOMAXCONN` ，在Linux内核2.4.25之后，在配置文件 `/proc/sys/net/core/somaxconn` 中直接修改，或者在 `/etc/sysctl.conf` 中配置 `net.core.somaxconn = 32768` 。

如果 `accpet queue` 队列满了，`server` 将发送一个 `ECONNREFUSED` 错误信息 `Connection refuse`到 `client` 。

立即生效还可以使用命令：`sysctl -w net.core.somaxconn=32768`。

要想永久生效，需要在文件 `/etc/sysctl.conf` 中增加一行：

`net.core.somaxconn = 32768`，

或者使用命令：

`echo "net.core.somaxconn = 32768" >> /etc/sysctl.conf`,

然后执行命令 `sysctl -p` 以生效。

注：Redis配置项 `tcp-backlog` 的值不能超过somaxconn的大小。

ss命令来显示:

```sh
[root@localhost ~]# ss -l
State       Recv-Q Send-Q                                     Local Address:Port                                         Peer Address:Port     
LISTEN      0      128                                                    *:http                                                    *:*       
LISTEN      0      128                                                   :::ssh                                                    :::*       
LISTEN      0      128                                                    *:ssh                                                     *:*       
LISTEN      0      100                                                  ::1:smtp                                                   :::*       
LISTEN      0      100                                            127.0.0.1:smtp                                                    *:*  
```

在LISTEN状态，其中 Send-Q 即为Accept queue的最大值，Recv-Q 则表示Accept queue中等待被服务器accept()。

客户端connect()返回不代表TCP连接建立成功，有可能此时 `accept queue` 已满，系统会直接丢弃后续ACK请求；客户端误以为连接已建立，开始调用等待至超时；服务器则等待ACK超时，会重传 `SYN+ACK` 给客户端，重传次数受限 `net.ipv4.tcp_synack_retries` ，默认为5，表示重发5次，每次等待30~40秒，即半连接默认时间大约为180秒，该参数可以在tcp被洪水攻击是临时启用这个参数。

**查看SYN queue 溢出**

```sh
[root@localhost ~]# netstat -s | grep LISTEN
102324 SYNs to LISTEN sockets dropped
```
　　
**查看Accept queue 溢出**

```sh
[root@localhost ~]# netstat -s | grep TCPBacklogDrop
TCPBacklogDrop: 2334
```

参考：

[TCP/IP协议中backlog参数](https://www.cnblogs.com/Orgliny/p/5780796.html)

[浅谈tcp socket的backlog参数](https://www.jianshu.com/p/e6f2036621f4)

[Linux TCP队列相关参数的总结](https://developer.aliyun.com/article/4252)