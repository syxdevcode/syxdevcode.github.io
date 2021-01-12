---
title: Linux系统实现高并发内核优化
date: 2020-09-15 11:50:00
tags:
- Linux
- CentOS7
- Linux优化
categories:
- Linux
---

## Linux 文件打开最大数

```sh
vi /etc/security/limits.conf
* soft nproc 65535
* hard nproc 65535
* soft nofile 65535
* hard nofile 65535
```

参考：[Linux最大文件打开数](https://syxdevcode.github.io/2020/08/04/Linux%E6%9C%80%E5%A4%A7%E6%96%87%E4%BB%B6%E6%89%93%E5%BC%80%E6%95%B0/)

## 内核优化

方法一：修改 `/proc` 下内核参数文件内容，不能使用编辑器来修改内核参数文件，理由是由于内核随时可能更改这些文件中的任意一个，另外，这些内核参数文件都是虚拟文件，实际中不存在，因此不能使用编辑器进行编辑，而是使用echo命令，然后从命令行将输出重定向至 `/proc` 下所选定的文件中。如：将 `timeout_timewait` 参数设置为30秒：

`echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout`

参数修改后立即生效，但是重启系统后，该参数又恢复成默认值。因此，想永久更改内核参数，需要修改/etc/sysctl.conf文件
 
方法二．修改 `/etc/sysctl.conf` 文件

优化适合`apache`，`nginx`，`squid` 多种等web应用。

文件：`/etc/sysctl.conf`

```sh
# 表示套接字由本端要求关闭，这个参数决定了它保持在FIN-wAIT-2状态的时间，默认值是60秒
# server端主动发起断开连接后保持在FIN-WAIT-2状态的时间（建议30s）
# 该参数对应系统路径为：/proc/sys/net/ipv4/tcp_fin_timeout 60
net.ipv4.tcp_fin_timeout = 30

## reuse和recycle这俩个参数是为防止生产环境下 web，nginx 等业务服务器 time_wait 网络状态数量过多设置的

# 表示开启重用，允许TIME-wAIT sockets重新用于新的TCP链接，默认值为0
# 表示关闭，该参数对应系统路径为：/proc/sys/net/ipv4/tcp_tw_reuse 0
net.ipv4.tcp_tw_reuse = 1

# # 表示开启TCP链接中TIME_WAIT sockets的快速回收，
# 该参数对应系统路径为：/proc/sys/net/ipv4/tcp_tw_recycle
# 默认为0 表示关闭，建议为0
net.ipv4.tcp_tw_recycle = 0
```

<details>
  <summary>reuse/recycle注解：</summary>

因为TCP连接是双向的，所以在关闭连接的时候，两个方向各自都需要关闭。先发 `FIN` 包的一方执行的是 `主动关闭`；后发 `FIN` 包的一方执行的是 `被动关闭`。主动关闭的一方会进入`TIME_WAIT` 状态，并且在此状态停留两倍的 `MSL` maximum segment lifetime(`最大分节生命期`或`最大报文存活时间`，这是一个IP数据包能在互联网上生存的最长时间，超过这个时间IP数据包将在网络中消失 。MSL在RFC 1122上建议是2分钟，而源自berkeley的TCP实现传统上使用30秒。一般Linux内核设置30秒 )时长。`TIME_WAIT` 状态维持时间是两个MSL时间长度，也就是在1-4分钟。Windows操作系统就是4分钟。

为什么主动方要傻乎乎等 `2MSL` 呢?不等,行不行?

TCP目的是可靠传输,主动关闭的一方发出 `FIN`,被动方回复`ACK`,接着被动方发送 `FIN` ,主动方收到被动关闭的一方发出的 `FIN` 包后，回应 `ACK` 包，同时进入 `TIME_WAIT` 状态。但是因为网络原因，主动关闭的一方发送的这个 `ACK` 包很可能延迟，从而触发被动连接一方重传 `FIN` 包。极端情况下，这一去( `ACK` 去被动方)一回(重传 `FIN` 回来)，就是两倍的 `MSL` 时长。

如果主动关闭的一方跳过 `TIME_WAIT` 直接进入 `CLOSED` ，或者在 `TIME_WAIT` 停留的时长不足两倍的 `MSL` ，那么当被动关闭的一方早先发出的 `FIN` 延迟包到达或者重传 `FIN` 包到达后，就可能出现类似下面的问题：

* 主动方旧的TCP连接已经不存在了，主动方只能返回RST包
* 主动方新的TCP连接被建立起来了，延迟包可能干扰新的连接

所以, `TIME_WAIT` 必须等, `2MSL` 不能少

**减少TIME_WAIT**

`TIME_WAIT` 期间,资源不会释放,现在都追求高性能高并发,快速释放资源是躲不掉的.对于客户端因为有端口 `65535` 问题，`TIME_WAIT` 过多直接影响处理能力. 对于服务器,无端口数量限制的问题,Linux优化也很给力,每个处于 `TIME_WAIT` 状态下连接内存消耗很少, 而且也能通过 `tcp_max_tw_buckets = ${你要的阈值}` 配置最大上限，但是对于短连接为主的web服务器,几十万的连接,基数很大,耗得内存也不小，快速释放总是好的。

**tcp_tw_recycle:回收TIME_WAIT连接**

对客户端和服务器同时起作用，开启后在 3.5*RTO 内回收，RTO 200ms~ 120s 具体时间视网络状况。RTO(Retransmission TimeOut)重传超时时间。内网状况比 `tcp_tw_reuse` 稍快，公网尤其移动网络大多要比 `tcp_tw_reuse` 慢，优点就是能够回收服务端的 `TIME_WAIT` 数量。

但是，需要注意的是:当多个客户端通过NAT方式联网并与服务端交互时，服务端看到的是同一个IP，也就是说对服务端而言这些客户端实际上等同于一个，可惜由于这些客户端的时间戳可能存在差异，于是乎从服务端的视角看，便可能出现时间戳错乱的现象，进而直接导致时间戳小的数据包被丢弃。客户端处于NAT很常见，基本公司家庭网络都走NAT。

`tcp_tw_reuse`:复用 `TIME_WAIT` 连接 只对客户端起作用,1秒后才能复用，当创建新连接的时候，如果可能的话会考虑复用相应的 `TIME_WAIT` 连接。通常认为 `tcp_tw_reuse `比`tcp_tw_recycle` 安全一些，这是因为一来 `TIME_WAIT` 创建时间必须超过一秒才可能会被复用；二来只有连接的时间戳是递增的时候才会被复用。

客户端请求服务器,服务器响应后主动关闭连接,`TIME_WAIT` 存在于服务器,服务器是被连接者,没有复用一说,所以只对客户端起作用.如果是客户端主动关闭, `TIME_WAIT` 存在于客户端,这个时候再次连接服务器,可以复用之前 `TIME_WAIT` 留下的半废品。

`tcp_timestamps`:以上两点,必须在客户端和服务端 `timestamps` 开启时才管用(默认开启) 需要根据 `timestamp` 的递增性来区分是否新连接

**总结**

客户端 `tcp_tw_reuse` 复用连接管用, `tcp_tw_recycle` 有用,但是客户端主要不是接受连接,用处不大
服务器 `tcp_tw_recycle` 回收连接管用,`tcp_tw_reuse` 复用无效.为了减少 `TIME_WAIT` 留在服务器,可以在服务器开启 `KeepAlive` ,尽量不让服务器主动关闭,而是客户端主动关闭,减少`TIME_WAIT` 产生。

</details>

```sh
# 表示开启SYN Cookies功能，当出现SYN等待队列溢出时，启用Cookies来处理，可防范少量SYN攻击，
# 该参数对应系统路径为：/proc/sys/net/ipv4/tcp_syncookies，默认为1，表示开启
net.ipv4.tcp_syncookies = 1

# 表示当keepalive启用时，TCP发送keepalive消息的频度，默认是2小时，建议更改为5分钟，
# 该参数对应系统路径为：/proc/sys/net/ipv4/tcp_keepalive_time，默认为7200秒
net.ipv4.tcp_keepalive_time = 300

# 该选项用来设定允许系统打开的端口范围，即用于向外链接的端口范围，
# 每个TCP客户端连接都要占用一个唯一的本地端口号(此端口号在系统的本地端口号范围限制中)，
# 如果现有的TCP客户端连接已将所有的本地端口号占满。将不能创建新的TCP连接。
# 该参数对应系统路径为：/proc/sys/net/ipv4/ip_local_port_range 默认，32768 60999
# 理论上单独一个进程最多可以同时建立60000多个TCP客户端连接。
net.ipv4.ip_local_port_range = 1024 65000

# 表示SYN队列的长度，默认为1024，建议加大队列的长度，为8182或更多，这样可以容纳更多等待链接的网络连接数，
# 该参数为服务器端用于记录那些尚未收到客户端确认信息的链接请求的最大值，
# 该参数对应系统路径为：/proc/sys/net/ipv4/tcp_max_syn_backlog
net.ipv4.tcp_max_syn_backlog = 262144

# 该选项默认值是128，这个参数用于调节系统同时发起的TCP连接数，
# 在高并发的请求中，默认的值可能会导致链接超时或重传，因此，需要结合并发请求数来调节此值，
# 该参数对应系统路径为：/proc/sys/net/ipv4/somaxconn 128   
# 默认没有这个配置，需要自己生成，somaxconn值不应超过65535
net.core.somaxconn = 32768
```

listen方法指定的backlog是在用户态指定的，内核态的参数优先级高于用户态的参数，所以即使在listen方法里面指定backlog是一个大于somaxconn的值，socket在内核态运行时还会检查一次somaxconn，如果连接数超过somaxconn就会等待。

**在没有调优的centOS7.4版本的服务器上，由于受到系统级别的限制，在该服务器上运行的服务端程序，在同一时间，最大只能接受128个客户端发起持久连接，并且只能处理128个客户端的数据通信。**

```sh
# 表示系统同时保持 TIME_WAIT 套接字的最大数量，如果超过这个数值，TIME_WAIT 套接字将立刻被清除并打印警告信息，
# 对于Aapache，Nginx等服务器来说可以将其调低一点，如改为 5000-30000，不用业务的服务器也可以给大一点，
# 比如LVS，Squid，该参数对应系统路径为：/proc/sys/net/ipv4/tcp_max_tw_buckets
# #1st低于此值,TCP没有内存压力,2nd进入内存压力阶段,3rdTCP拒绝分配socket(单位：内存页)
net.ipv4.tcp_max_tw_buckets = 30000

# 外向syn握手重试次数
# 该参数对应系统路径为：/proc/sys/net/ipv4/tcp_syn_retries，默认是6
net.ipv4.tcp_syn_retries = 1

# 为了打开对端的连接，内核需要发送一个 SYN 并附带一个回应前面一个 SYN 的 ACK。也就是所谓三次握手中的第二次握手。
# 该参数对应系统路径为：/proc/sys/net/ipv4/tcp_synack_retries，默认是5
# syn-ack握手状态重试次数，默认5，遭受SYN Flood [(SYN洪水) 是种典型的DoS (Denial of Service，拒绝服务)]攻击时改为1或2
net.ipv4.tcp_synack_retries = 1

# 网卡设备将请求放入队列的长度
# 表示当每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许发送到队列的数据包最大数，
# 该参数对应系统路径为：/proc/sys/net/ipv4/netdev_max_backlog
# 默认没有这个配置，需要自己生成
net.core.netdev_max_backlog = 262144

# 用于设定系统中最多有多少个TCP套接字不被关联到任何一个用户文件句柄上，
# 如果超过这个数值，孤立链接将立即被复位并打印出警号信息，这个限制只是为了防止简单的DoS攻击，不能过分依靠这个限制甚至
# 人为减小这个值，更多的情况是增加这个值，该参数对应系统路径为：/proc/sys/net/ipv4/tcp_max_orphans
net.ipv4.tcp_max_orphans = 3276800

#开启窗口缩放功能，要支持超过64KB的TCP窗口，必须启用该值,TCP连接双方都启用时才生效
net.ipv4.tcp_window_scaling = 1

# 表示套接字接收缓冲区的内存最大值
net.core.rmem_max = 16777216

# 表示套接字发送缓冲区大小的最大值,会覆盖 net.ipv4.tcp_wmem的MAX值
net.core.wmem_max = 16777216

# 接受缓冲的大小:MIN，DEFAULT,MAX
# 即TCP读取缓冲区，单位为字节byte
# net.ipv4.tcp_mem[0]:低于此值，TCP没有内存压力。
# net.ipv4.tcp_mem[1]:在此值下，进入内存压力阶段。
# net.ipv4.tcp_mem[2]:高于此值，TCP拒绝分配socket。
net.ipv4.tcp_rmem = 4096  87380 16777216

# socket的发送缓存区分配的MIN，DEFAULT,MAX
# 发送缓冲区，单位是字节byte
net.ipv4.tcp_wmem = 4096 16384 16777216

# 1低于此值,TCP没有内存压力,2在此值下,进入内存压力阶段，3高于此值,TCP拒绝分配socket.
# 内存单位是页，1页等于4096字节，1 Page = 4096 Bytes
# 32GB内存机器建议：8G  12G  16G
net.ipv4.tcp_mem = 2097152 3145728 4194304

# 禁用时间戳，时间戳可以避免序列号的卷绕
net.ipv4.tcp_timestamps = 0

# 定义SYN重试次数
net.ipv4.tcp_sack = 1

# 单独一个进程最多可以同时建立20000多个TCP客户端连接
# 不建议自行设置
net.ipv4.ip_conntrack_max = 20000

```

/etc/sysctl.conf

```sh
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_syncookies = 1
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_max_syn_backlog = 262144
net.core.somaxconn = 65535
net.ipv4.tcp_max_tw_buckets = 30000
net.ipv4.tcp_syn_retries = 1
net.ipv4.tcp_synack_retries = 1
net.core.netdev_max_backlog = 262144
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.tcp_window_scaling = 1
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 16384 16777216
net.ipv4.tcp_mem = 2097152 3145728 4194304
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_timestamps = 0
net.ipv4.tcp_sack = 1
```

查看系统级文件句柄修改，是否生效

```sh
sysctl -p
```

备忘录：

```sh
# 查看当前的连接数状况
# SYN_RECV表示正在等待处理的请求数；ESTABLISHED表示正常数据传输状态；TIME_WAIT表示处理完毕，等待超时结束的请求数
netstat -nat|awk '{print awk $NF}'|sort|uniq -c|sort -n

# 查看IP连接数状况
netstat -nat|grep ":80"|awk '{print $5}' |awk -F: '{print $1}' | sort| uniq -c|sort -n

# 查看活动连接数
ss -n | grep ESTAB | wc -l

# 统计当前各种状态的连接的数量的命令
netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'

# 返回结果如下：
LAST_ACK 14
SYN_RECV 348
ESTABLISHED 70
FIN_WAIT1 229
FIN_WAIT2 30
CLOSING 33
TIME_WAIT 18122

# 对上述结果的解释：
CLOSED：无连接是活动的或正在进行
LISTEN：服务器在等待进入呼叫
SYN_RECV：一个连接请求已经到达，等待确认
SYN_SENT：应用已经开始，打开一个连接
ESTABLISHED：正常数据传输状态
FIN_WAIT1：应用说它已经完成
FIN_WAIT2：另一边已同意释放
ITMED_WAIT：等待所有分组死掉
CLOSING：两边同时尝试关闭
TIME_WAIT：另一边已初始化一个释放
LAST_ACK：等待所有分组死掉

# 获取当前socket连接状态统计信息
cat /proc/net/sockstat
```

参考：

[Linux内核 TCP/IP、Socket参数调优](https://www.cnblogs.com/zengkefu/p/5749009.html)

[Linux系统优化实现高并发](https://www.cnblogs.com/liluxiang/p/9318493.html)

[Linux内核优化](https://www.cnblogs.com/augusite/p/10774014.html)

[Linux 下高并发系统内核优化](https://www.cnblogs.com/heping1314/p/11214638.html)

[centos7之系统优化方案](https://www.cnblogs.com/jokerbj/p/9133093.html)

[Linux 内核优化-调大TCP最大连接数](https://www.jianshu.com/p/fa35d91b727b)