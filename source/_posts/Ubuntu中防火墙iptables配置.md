---
title: Ubuntu中防火墙iptables配置
date: 2022-09-08 15:02:50
tags:
  - Ubuntu
  - Linux基础
  - Linux
  - iptables
  - 安全策略
categories:
  - 防火墙
---

## 查看系统是否安装防火墙

```shell
which iptables
```

如果没有安装，那么使用 `sudo apt-get install iptables` 安装。

## 查看防火墙的配置信息

```shell
sudo iptables -L
```

<!--more-->

## 新建规则文件

mkdir /etc/iptables #先新建目录，本身无此目录
vim /etc/iptables/rules.v4

rules.v4 示例：

```shell
*filter
:INPUT DROP [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
:syn-flood - [0:0]
-A INPUT -i lo -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 80 -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
-A INPUT -p icmp -m limit --limit 100/sec --limit-burst 100 -j ACCEPT
-A INPUT -p icmp -m limit --limit 1/s --limit-burst 10 -j ACCEPT
-A INPUT -p tcp -m tcp --tcp-flags FIN,SYN,RST,ACK SYN -j syn-flood
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A syn-flood -p tcp -m limit --limit 3/sec --limit-burst 6 -j RETURN
-A syn-flood -j REJECT --reject-with icmp-port-unreachable
COMMIT
```

## 使防火墙生效

```shell
iptables-restore < /etc/iptables/rules.v4

# 创建文件，添加以下内容，使防火墙开机启动
vim /etc/network/if-pre-up.d/iptables

# iptables：内容
#!/bin/bash
iptables-restore < /etc/iptables/rules.v4

# 添加执行权限
chmod +x /etc/network/if-pre-up.d/iptables

#查看规则是否生效
iptables -L -n
```

## iptables 常用命令

```shell
# 列出所有iptables规则
iptables -nvL

# 列出所有iptables规则
iptables -L -n

# 列出所有iptables规则，并显示编号【有编号才好删】
iptables -L -n --line-number

# 将当前iptables规则保存到指定文件中
iptables-save > 文件绝对路径

# 从指定文件中加载iptables规则
iptables-restore < 文件绝对路径
```

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

## 实例

```sh
# 清除现有规则
iptables -F
iptables -t nat -F

# 设置默认策略
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT

# 设置 NAT 表
iptables -t nat -A POSTROUTING -o docker0 -j MASQUERADE
iptables -t nat -A POSTROUTING ! -o docker0 -s 10.10.13.0/24 -j MASQUERADE
iptables -t nat -A POSTROUTING ! -o docker0 -s 10.10.14.0/24 -j MASQUERADE

# 设置 filter 表
iptables -A FORWARD -i docker0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i eth0 -o docker0 -j ACCEPT

# 忽略 lo 接口的流量
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# 允许已建立的连接
iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A OUTPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

# 拒绝无效的包
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
iptables -A OUTPUT -m conntrack --ctstate INVALID -j DROP
iptables -A FORWARD -m conntrack --ctstate INVALID -j DROP

# 保存规则
iptables-save > /etc/iptables/rules.v4
```
