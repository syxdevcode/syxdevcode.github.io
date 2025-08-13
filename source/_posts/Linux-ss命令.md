---
title: Linux ss命令
date: 2021-03-29 15:04:56
tags:
- Linux
- CentOS7
- Linux优化
- Linux基础命令
categories:
- Linux基础命令
---

ss命令用来显示处于活动状态的套接字信息。ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。ss命令是Linux CentOS 7中iproute软件包的一部分，默认已经安装。

当服务器的socket连接数量变得非常大时，无论是使用netstat命令还是直接cat /proc/net/tcp，执行速度都会很慢。

ss快的秘诀在于，它利用到了TCP协议栈中tcp_diag。tcp_diag是一个用于分析统计的模块，可以获得Linux 内核中第一手的信息，这就确保了ss的快捷高效。当然，如果你的系统中没有tcp_diag，ss也可以正常运行，只是效率会变得稍慢。

```sh
yum -y install iproute
yum info iproute
```

语法

`ss(选项)`

选项

* -h：显示帮助信息；
* -V：显示指令版本信息；
* -n：不解析服务名称，以数字方式显示；
* -a：显示所有的套接字；
* -l：显示处于监听状态的套接字；
* -o：显示计时器信息；
* -m：显示套接字的内存使用情况；
* -p：显示使用套接字的进程信息；
* -i：显示内部的TCP信息；
* -4：只显示ipv4的套接字；
* -6：只显示ipv6的套接字；
* -t：只显示tcp套接字；
* -u：只显示udp套接字；
* -d：只显示DCCP套接字；
* -w：仅显示RAW套接字；
* -x：仅显示UNIX域套接字。

## 实例

```sh
# 显示ICP连接
ss -t -a

# 显示 Sockets 摘要
ss -s

# 列出所有打开的网络连接端口
ss -l

# 查看进程使用的socket
ss -pl

# 显示所有UDP Sockets
ss -u -a

# 查看6379端口
ss -ta sport = :6379 | head
ss -pl | grep 3306

# 显示所有状态为established的SMTP连接
ss -o state established '( dport = :smtp or sport = :smtp )' 

# 显示所有状态为Established的HTTP连接
ss -o state established '( dport = :http or sport = :http )' 

# 列举出处于 FIN-WAIT-1状态的源端口为 80或者 443，目标网络为 193.233.7/24所有 tcp套接字
ss -o state fin-wait-1 '( sport = :http or sport = :https )' dst 193.233.7/24

# 用TCP 状态过滤Sockets
ss -4 state closing 
ss -4 state FILTER-NAME
ss -6 state FILTER-NAME

# FILTER-NAME-HERE 可以代表以下任何一个：

established
syn-sent
syn-recv
fin-wait-1
fin-wait-2
time-wait
closed
close-wait
last-ack
listen
closing
all : 所有以上状态
connected : 除了listen and closed的所有状态
synchronized :所有已连接的状态除了syn-sent
bucket : 显示状态为maintained as minisockets,如：time-wait和syn-recv.
big : 和bucket相反.

# 匹配远程地址和端口号
ss dst ADDRESS_PATTERN
ss dst 192.168.1.5
ss dst 192.168.119.113:http 
ss dst 192.168.119.113:smtp 
ss dst 192.168.119.113:443

# 匹配本地地址和端口号
ss src ADDRESS_PATTERN
ss src 192.168.119.103
ss src 192.168.119.103:http
ss src 192.168.119.103:80
ss src 192.168.119.103:smtp
ss src 192.168.119.103:25

# 将本地或者远程端口和一个数比较
ss dport OP PORT 
ss sport OP PORT

# ss dport OP PORT 远程端口和一个数比较；
# ss sport OP PORT 本地端口和一个数比较。
OP 可以代表以下任意一个: 
<= or le : 小于或等于端口号
>= or ge : 大于或等于端口号
== or eq : 等于端口号
!= or ne : 不等于端口号
< or gt : 小于端口号
> or lt : 大于端口号

# ss 和 netstat 效率对比
time netstat -at
time ss
```

参考：

[ss命令](https://man.linuxde.net/ss)
[每天一个linux命令（57）：ss命令](https://www.cnblogs.com/peida/archive/2013/03/11/2953420.html)