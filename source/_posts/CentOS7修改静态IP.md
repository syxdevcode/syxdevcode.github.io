---
title: 'CentOS7修改静态IP'
date: 2017-11-30 08:30:10
tags:
- Linux
- CentOS7
categories:
- CentOS7
---
# CentOS7修改静态IP

## 进入终端

```linux
[root@localhost toor]# cd /etc/sysconfig/network-scripts
[root@localhost network-scripts]# ls
ifcfg-eno16777736         ifdown-isdn      ifup-eth     ifup-Team
ifcfg-eno16777736-1       ifdown-post      ifup-ib      ifup-TeamPort
ifcfg-eno16777736.6BOP8P  ifdown-ppp       ifup-ippp    ifup-tunnel
ifcfg-lo                  ifdown-routes    ifup-ipv6    ifup-wireless
ifcfg-Wired_connection_1  ifdown-sit       ifup-isdn    init.ipv6-global
ifdown                    ifdown-Team      ifup-plip    network-functions
ifdown-bnep               ifdown-TeamPort  ifup-plusb   network-functions-ipv6
ifdown-eth                ifdown-tunnel    ifup-post    route-eno16777736-1
ifdown-ib                 ifup             ifup-ppp
ifdown-ippp               ifup-aliases     ifup-routes
ifdown-ipv6               ifup-bnep        ifup-sit
[root@localhost network-scripts]# vim ifcfg-eno16777736
```

示例：

```linux
TYPE=Ethernet
BOOTPROTO=static #dhcp改为static
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=eno16777736
UUID=7b7f487c-4a01-464b-9587-dae9daca6bc9
DEVICE=eno16777736
ONBOOT=yes
IPADDR=192.168.0.122 #静态IP
GATEWAY=192.168.0.1 #默认网关
NETMASK=255.255.255.0 #子网掩码
```

重启网络服务

```linux
service network restart
```

若出现重启失败的话，可以试着把ifcfg-eno文件里的DEVICE一行删除试试