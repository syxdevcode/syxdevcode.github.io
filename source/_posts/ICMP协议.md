---
title: ICMP协议
date: 2023-02-07 10:08:57
tags:
  - TCP协议
  - IP网络
  - ICMP协议
categories:
  - ICMP协议
---


&emsp;&emsp;ICMP（Internet Control Message Protocol）Internet控制报文协议。它是TCP/IP协议簇的一个子协议，用于在IP主机、路由器之间传递控制消息。控制消息是指网络通不通、主机是否可达、路由是否可用等网络本身的消息。这些控制消息虽然并不传输用户数据，但是对于用户数据的传递起着重要的作用。

&emsp;&emsp;ICMP使用IP的基本支持，就像它是一个更高级别的协议，但是，ICMP实际上是IP的一个组成部分，必须由每个IP模块实现。

## 简介

&emsp;&emsp;ICMP协议是一种面向无连接的协议，用于传输出错报告控制信息。它是一个非常重要的协议，它对于网络安全具有极其重要的意义。它属于网络层协议，主要用于在主机与路由器之间传递控制信息，包括报告错误、交换受限控制和状态信息等。当遇到IP数据无法访问目标、IP路由器无法按当前的传输速率转发数据包等情况时，会自动发送ICMP消息。

&emsp;&emsp;ICMP 是 TCP/IP 模型中网络层的重要成员，与 IP 协议、ARP 协议、RARP 协议及 IGMP 协议共同构成 TCP/IP 模型中的网络层。ping 和 tracert是两个常用网络管理命令，ping 用来测试网络可达性，tracert 用来显示到达目的主机的路径。ping和 tracert 都利用 ICMP 协议来实现网络功能，它们是把网络协议应用到日常网络管理的典型实例。
&emsp;&emsp;从技术角度来说，ICMP就是一个“错误侦测与回报机制”，其目的就是让我们能够检测网路的连线状况﹐也能确保连线的准确性。当路由器在处理一个数据包的过程中发生了意外，可以通过ICMP向数据包的源端报告有关事件。

&emsp;&emsp;其功能主要有：侦测远端主机是否存在，建立及维护路由资料，重导资料传送路径（ICMP重定向），资料流量控制。ICMP在沟通之中，主要是透过不同的类别(Type)与代码(Code) 让机器来识别不同的连线状况。

&emsp;&emsp;ICMP 是个非常有用的协议﹐尤其是当我们要对网路连接状况进行判断的时候。

<!--more-->
## 工作原理

&emsp;&emsp;ICMP提供一致易懂的出错报告信息。发送的出错报文返回到发送原数据的设备，因为只有发送设备才是出错报文的逻辑接受者。发送设备随后可根据ICMP报文确定发生错误的类型，并确定如何才能更好地重发失败的数据包。但是ICMP唯一的功能是报告问题而不是纠正错误，纠正错误的任务由发送方完成。

&emsp;&emsp;我们在网络中经常会使用到ICMP协议，比如我们经常使用的用于检查网络通不通的Ping命令（Linux和Windows中均有），这个“Ping”的过程实际上就是ICMP协议工作的过程。还有其他的网络命令如跟踪路由的Tracert命令也是基于ICMP协议的。

## 报文格式

&emsp;&emsp;ICMP报文包含在IP数据报中，属于IP的一个用户，IP头部就在ICMP报文的前面，所以一个ICMP报文包括IP头部、ICMP头部和ICMP报文，IP头部的Protocol值为1就说明这是一个ICMP报文，ICMP头部中的类型（Type）域用于说明ICMP报文的作用及格式，此外还有一个代码（Code）域用于详细说明某种ICMP报文的类型，所有数据都在ICMP头部后面。

ICMP报文格式具体由RFC 777  ，RFC 792 规范

## ICMP类型

已经定义的ICMP消息类型大约有10多种，每种ICMP数据类型都被封装在一个IP数据包中。主要的ICMP消息类型包括以下几种。

### 响应请求

我们日常使用最多的ping，就是响应请求（Type=8）和应答（Type=0），一台主机向一个节点发送一个Type=8的ICMP报文，如果途中没有异常（例如被路由器丢弃、目标不回应ICMP或传输失败），则目标返回Type=0的ICMP报文，说明这台主机存在，更详细的tracert通过计算ICMP报文通过的节点来确定主机与目标之间的网络距离。

### 目标不可到达、源抑制和超时报文

这三种报文的格式是一样的，目标不可到达报文（Type=3）在路由器或主机不能传递数据报时使用，例如我们要连接对方一个不存在的系统端口（端口号小于1024）时，将返回Type=3、Code=3的ICMP报文，它要告诉我们：“嘿，别连接了，我不在家的！”，常见的不可到达类型还有网络不可到达（Code=0）、主机不可到达（Code=1）、协议不可到达（Code=2）等。源抑制则充当一个控制流量的角色，它通知主机减少数据报流量，由于ICMP没有恢复传输的报文，所以只要停止该报文，主机就会逐渐恢复传输速率。最后，无连接方式网络的问题就是数据报会丢失，或者长时间在网络游荡而找不到目标，或者拥塞导致主机在规定时间内无法重组数据报分段，这时就要触发ICMP超时报文的产生。超时报文的代码域有两种取值：Code=0表示传输超时，Code=1表示重组分段超时。

### 时间戳

时间戳请求报文（Type=13）和时间戳应答报文（Type=14）用于测试两台主机之间数据报来回一次的传输时间。传输时，主机填充原始时间戳，接收方收到请求后填充接收时间戳后以Type=14的报文格式返回，发送方计算这个时间差。一些系统不响应这种报文。 

### 全部消息类型

|  TYPE  |  CODE  |  Description  |  Query  |  Error  |
|--------|--------|---------------|---------|---------|
| 0 | 0  | Echo Reply——回显应答（Ping应答）                               | x |        |
| 3 | 0  | Network Unreachable——网络不可达                               |  |   x    |
| 3 | 1  | Host Unreachable——主机不可达                                  |  |   x    |
| 3 | 2  |Protocol Unreachable——协议不可达                               |  |   x    |
| 3 | 3  | Port Unreachable——端口不可达                                  |  |   x    |
| 3 | 4  | Fragmentation needed but no frag. bit set——需要进行分片但设置不分片比特 |  | x |
| 3 | 5  | Source routing failed——源站选路失败                            |   | x |
| 3 | 6  | Destination network unknown——目的网络未知                      |   | x |
| 3 | 7  | Destination host unknown——目的主机未知                         |   | x |
| 3 | 8  | Source host isolated (obsolete)——源主机被隔离（作废不用）       |   | x |
| 3 | 9  | Destination network administratively prohibited——目的网络被强制禁止 |   | x |
| 3 | 10 | Destination host administratively prohibited——目的主机被强制禁止    |   | x |
| 3 | 11 | Network unreachable for TOS——由于服务类型TOS，网络不可达             |   | x |
| 3 | 12 | Host unreachable for TOS——由于服务类型TOS，主机不可达                |   | x |
| 3 | 13 | Communication administratively prohibited by filtering——由于过滤，通信被强制禁止 |   | x |
| 3 | 14 | Host precedence violation——主机越权                                 |   | x |
| 3 | 15 | Precedence cutoff in effect——优先中止生效                            |   | x |
| 4 | 0  | Source quench——源端被关闭（基本流控制）                               |   |   |
| 5 | 0  | Redirect for network——对网络重定向                                   |   |   |
| 5 | 1  | Redirect for host——对主机重定向                                      |   |   |
| 5 | 2  | Redirect for TOS and network——对服务类型和网络重定向                  |   |   |
| 5 | 3  | Redirect for TOS and host——对服务类型和主机重定向                     |   |   |
| 8 | 0  | Echo request——回显请求（Ping请求）                                   | x  |   |
| 9 | 0  | Router advertisement——路由器通告                                     |   |   |
| 10 | 0 | Route solicitation——路由器请求                                       |   |   |
| 11 | 0 | TTL equals 0 during transit——传输期间生存时间为0                      |   | x |
| 11 | 1 | TTL equals 0 during reassembly——在数据报组装期间生存时间为0            |   | x |
| 12 | 0 | IP header bad (catchall error)——坏的IP首部（包括各种差错）             |   | x |
| 12 | 1 | Required options missing——缺少必需的选项                              |   | x |
| 13 | 0 | Timestamp request (obsolete)——时间戳请求（作废不用）                   | x |   |
| 14 |   | Timestamp reply (obsolete)——时间戳应答（作废不用）                     | x  |   |
| 15 | 0 | Information request (obsolete)——信息请求（作废不用）                   | x  |   |
| 16 | 0 | Information reply (obsolete)——信息应答（作废不用）                     | x  |   |
| 17 | 0 | Address mask request——地址掩码请求                                    | x |    |
| 18 | 0 | Address mask reply——地址掩码应答                                      |   |   |

## 应用

ICMP 协议应用在许多网络管理命令中，下面以 ping 和 tracert 命令为例详细介绍 ICMP 协议的应用。

### （1） ping 命令使用 ICMP 回送请求和应答报文

在网络可达性测试中使用的分组网间探测命令 ping 能产生 ICMP 回送请求和应答报文。目的主机收到 ICMP 回送请求报文后立刻回送应答报文，若源主机能收到 ICMP 回送应答报文，则说明到达该主机的网络正常。

### （2）路由分析诊断程序 tracert 使用了 ICMP时间超过报文

tracert 命令主要用来显示数据包到达目的主机所经过的路径。通过执行一个 tracert 到对方主机的命令，返回数据包到达目的主机所经历的路径详细信息，并显示每个路径所消耗的时间。

参考：

[ICMP](https://baike.baidu.com/item/ICMP/572452)