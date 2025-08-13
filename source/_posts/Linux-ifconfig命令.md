---
title: Linux ifconfig命令
date: 2021-04-02 17:18:11
tags:
- Linux
- CentOS7
- Linux基础命令
- 网络基础
- Linux网络
categories:
- Linux基础命令
---

ifconfig 工具不仅可以被用来简单地获取网络接口配置信息，还可以修改这些配置。注意：用ifconfig命令配置的网卡信息，在网卡重启后机器重启后，配置就不存在。
<!--more-->
## 语法

```sh
ifconfig [网络设备][down up -allmulti -arp -promisc][add<地址>][del<地址>][<hw<网络设备类型><硬件地址>]
[io_addr<I/O地址>][irq<IRQ地址>][media<网络媒介类型>][mem_start<内存地址>][metric<数目>][mtu<字节>]
[netmask<子网掩码>][tunnel<地址>][-broadcast<地址>][-pointopoint<地址>][IP地址]
```

## 参数说明

* add<地址> 设置网络设备IPv6的IP地址。
* del<地址> 删除网络设备IPv6的IP地址。
* down 关闭指定的网络设备。
* <hw<网络设备类型><硬件地址> 设置网络设备的类型与硬件地址。
* io_addr<I/O地址> 设置网络设备的I/O地址。
* irq<IRQ地址> 设置网络设备的IRQ。
* media<网络媒介类型> 设置网络设备的媒介类型。
* mem_start<内存地址> 设置网络设备在主内存所占用的起始地址。
* metric<数目> 指定在计算数据包的转送次数时，所要加上的数目。
* mtu<字节> 设置网络设备的MTU。
* netmask<子网掩码> 设置网络设备的子网掩码。
* tunnel<地址> 建立IPv4与IPv6之间的隧道通信地址。
* up 启动指定的网络设备。
* -broadcast<地址> 将要送往指定地址的数据包当成广播数据包来处理。
* -pointopoint<地址> 与指定地址的网络设备建立直接连线，此模式具有保密功能。
* -promisc 关闭或启动指定网络设备的promiscuous模式。
* [IP地址] 指定网络设备的IP地址。
* [网络设备] 指定网络设备的名称。

## 返回字段说明

```sh
ifconfig -a
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.125.30.82  netmask 255.255.255.0  broadcast 192.125.30.255
        inet6 fe80::f816:3eff:fe11:cfb5  prefixlen 64  scopeid 0x20<link>
        ether fa:16:3e:11:cf:b5  txqueuelen 1000  (Ethernet)
        RX packets 17565972  bytes 3031241096 (2.8 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 9364501  bytes 1697699161 (1.5 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

 
# UP：表示“接口已启用”。
# BROADCAST ：表示“主机支持广播”。
# RUNNING：表示“接口在工作中”。
# MULTICAST：表示“主机支持多播”。
# LOOPBACK:回环,Loopback接口是一个特殊的虚拟网卡
# MTU:1500（最大传输单元）：1500字节
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
 
 
# inet ：网卡的IP地址。
# netmask ：网络掩码。
# broadcast ：广播地址。
inet 192.125.30.82  netmask 255.255.255.0  broadcast 192.125.30.255
 
 
# 网卡的IPv6地址
inet6 fe80::f816:3eff:fe11:cfb5  prefixlen 64  scopeid 0x20<link>
 
# 连接类型：Ethernet (以太网) HWaddr (硬件mac地址)
# txqueuelen (网卡设置的传送队列长度)
ether fa:16:3e:11:cf:b5  txqueuelen 1000  (Ethernet)
 
 
# RX packets 接收时，正确的数据包数。
# RX bytes 接收的数据量。
# RX errors 接收时，产生错误的数据包数。
# RX dropped 接收时，丢弃的数据包数。
# RX overruns 接收时，由于速度过快而丢失的数据包数。
# RX frame 接收时，发生frame错误而丢失的数据包数。
RX packets 17565972  bytes 3031241096 (2.8 GiB)
RX errors 0  dropped 0  overruns 0  frame 0
 
 
 
# TX packets 发送时，正确的数据包数。
# TX bytes 发送的数据量。
# TX errors 发送时，产生错误的数据包数。
# TX dropped 发送时，丢弃的数据包数。
# TX overruns 发送时，由于速度过快而丢失的数据包数。
# TX carrier 发送时，发生carrier错误而丢失的数据包数。
# collisions 冲突信息包的数目。
TX packets 9364501  bytes 1697699161 (1.5 GiB)
TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```