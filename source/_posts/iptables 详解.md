---
title: iptables 详解
date: 2022-09-08 15:57:51
tags:
  - Ubuntu
  - Linux基础
  - Linux
  - ufw
  - iptables
  - 安全策略
categories:
  - 防火墙
---

## iptables 的结构

iptables 是 Linux 防火墙系统的重要组成部分，iptables 的主要功能是实现对网络数据包进出设备及转发的控制。当数据包需要进入设备、从设备中流出或者由该设备转发、路由时，都可以使用 iptables 进行控制。

iptables 是集成在 Linux 内核中的包过滤防火墙系统。使用 iptables 可以添加、删除具体的过滤规则，iptables 默认维护着 4 个表和 5 个链，所有的防火墙策略规则都被分别写入这些表与链中。

## iptables 的表 tables 与链 chains

“四表”是指 iptables 的功能，默认的 iptables 规则表有 filter 表（过滤规则表）、nat 表（地址转换规则表）、mangle（修改数据标记位规则表）、raw（跟踪数据表规则表）：

- filter 表：控制数据包是否允许进出及转发，可以控制的链路有 INPUT、FORWARD 和 OUTPUT。
- nat 表：控制数据包中地址转换，可以控制的链路有 PREROUTING、INPUT、OUTPUT 和 POSTROUTING。
- mangle：修改数据包中的原数据，可以控制的链路有 PREROUTING、INPUT、OUTPUT、FORWARD 和 POSTROUTING。
- raw：控制 nat 表中连接追踪机制的启用状况，可以控制的链路有 PREROUTING、OUTPUT。

“五链”是指内核中控制网络的 NetFilter 定义的 5 个规则链。每个规则表中包含多个数据链：INPUT（入站数据过滤）、OUTPUT（出站数据过滤）、FORWARD（转发数据过滤）、PREROUTING（路由前过滤）和 POSTROUTING（路由后过滤），防火墙规则需要写入到这些具体的数据链中。

- 如果是外部主机发送数据包给防火墙本机，数据将会经过 PREROUTING 链与 INPUT 链；
- 如果是防火墙本机发送数据包到外部主机，数据将会经过 OUTPUT 链与 POSTROUTING 链；
- 如果防火墙作为路由负责转发数据，则数据将经过 PREROUTING 链、FORWARD 链以及 POSTROUTING 链。

<!--more-->

### Filter 表

Filter 是 iptables 的默认表,控制数据包是否允许进出及转发，它有以下三种内建链(chains)：

- INPUT 链 – 处理来自外部的数据。
- OUTPUT 链 – 处理向外发送的数据。
- FORWARD 链 – 将数据转发到本机的其他网卡设备上。

<!--more-->

### NAT 表

NAT 控制数据包中地址转换，表有三种内建链：

- PREROUTING 链 – 处理刚到达本机并在路由转发前的数据包。它会转换数据包中的目标 IP 地址（destination ip address），通常用于 DNAT(destination NAT)。
- POSTROUTING 链 – 处理即将离开本机的数据包。它会转换数据包中的源 IP 地址（source ip address），通常用于 SNAT（source NAT）。
- OUTPUT 链 – 处理本机产生的数据包。

### Mangle 表

Mangle 表用于指定如何处理数据包。它能改变 TCP 头中的 QoS 位。Mangle 表具有 5 个内建链（chains）：

- PREROUTING
- OUTPUT
- FORWARD
- INPUT
- POSTROUTING

### Raw 表

**Raw 表用于处理异常，它具有 2 个内建链:**

- PREROUTING chain
- OUTPUT chain

## IPTABLES 规则(Rules)

规则的关键知识点：

- Rules 包括一个条件和一个目标(target)
- 如果满足条件，就执行目标(target)中的规则或者特定值。
- 如果不满足条件，就判断下一条 Rules。

### 目标值（Target Values）

在 target 里指定的特殊值：

- ACCEPT – 允许防火墙接收数据包
- DROP – 防火墙丢弃包
- QUEUE – 防火墙将数据包移交到用户空间
- RETURN – 防火墙停止执行当前链中的后续 Rules，并返回到调用链(the calling chain)中。

```shell
# 查看各表中的规则命令
iptables -t filter --list

# 查看 mangle 表：

iptables -t mangle --list

# 查看 NAT 表：

iptables -t nat --list

# 查看 RAW 表：

iptables -t raw --list
```

以下例子表明在 filter 表的 input 链, forward 链, output 链中存在规则：

```shell
# iptables --list

Chain INPUT (policy ACCEPT)
num target prot opt source destination
1 RH-Firewall-1-INPUT all -- 0.0.0.0/0 0.0.0.0/0

Chain FORWARD (policy ACCEPT)
num target prot opt source destination
1 RH-Firewall-1-INPUT all -- 0.0.0.0/0 0.0.0.0/0

Chain OUTPUT (policy ACCEPT)
num target prot opt source destination

Chain RH-Firewall-1-INPUT (2 references)
num target prot opt source destination
1 ACCEPT all -- 0.0.0.0/0 0.0.0.0/0
2 ACCEPT icmp -- 0.0.0.0/0 0.0.0.0/0 icmp type 255
3 ACCEPT esp -- 0.0.0.0/0 0.0.0.0/0
4 ACCEPT ah -- 0.0.0.0/0 0.0.0.0/0
5 ACCEPT udp -- 0.0.0.0/0 224.0.0.251 udp dpt:5353
6 ACCEPT udp -- 0.0.0.0/0 0.0.0.0/0 udp dpt:631
7 ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:631
8 ACCEPT all -- 0.0.0.0/0 0.0.0.0/0 state RELATED,ESTABLISHED
9 ACCEPT tcp -- 0.0.0.0/0 0.0.0.0/0 state NEW tcp dpt:22
10 REJECT all -- 0.0.0.0/0 0.0.0.0/0 reject-with icmp-host-prohibited
```

以上输出包含下列字段：

num – 指定链中的规则编号
target – 前面提到的 target 的特殊值 prot – 协议：tcp, udp, icmp 等 source – 数据包的源 IP 地址 destination – 数据包的目标 IP 地址

## 清空所有 iptables 规则

在配置 iptables 之前，你通常需要用 iptables --list 命令或者 iptables-save 命令查看有无现存规则，因为有时需要删除现有的 iptables 规则：

```shell
iptables --flush
# 或者
iptables -F
```

下面命令是清除 iptables nat 表规则。

```shell
iptables -t nat -F
```

默认的 iptables 防火墙规则会立刻生效，但如果不保存，当计算机重启后所有的规则都会丢失，所以对防火墙规则进行及时保存的操作是非常必要的。

参考：

[Linux iptables 命令详解](https://zhuanlan.zhihu.com/p/513470164)

[iptables 详解](https://zhuanlan.zhihu.com/p/441089738)
