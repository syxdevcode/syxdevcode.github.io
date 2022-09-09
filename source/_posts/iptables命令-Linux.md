---
title: iptables命令-Linux
date: 2020-08-17 10:34:07
tags:
  - Linux
  - CentOS7
  - Ubuntu
  - Linux基础命令
  - iptables
  - 安全策略
categories:
  - Linux基础命令
---

## 简介

iptables 命令是 Linux 上常用的防火墙软件，是 netfilter 项目的一部分。可以直接配置，也可以通过许多前端和图形界面配置。

查看 iptables:

```shell
which iptables
whereis iptables
```

内核会按照顺序依次检查 iptables 防火墙规则:

- 如果发现有匹配的规则目录，则立刻执行相关动作，停止继续向下查找规则目录；
- 如果所有的防火墙规则都未能匹配成功，则按照默认策略处理。

使用 -A 选项添加防火墙规则会将该规则追加到整个链的最后，
使用 -I 选项添加的防火墙规则则会默认插入到链中作为第一条规则。

**语法:**

iptables (选项) (参数)

**选项:**

```sh
-t<表>：指定要操纵的表；
-A：向规则链中添加条目；
-D：从规则链中删除条目；
-I：向规则链中插入条目；
-R：替换规则链中的条目；
-L：表示查看当前表的所有规则，默认查看的是 filter 表，如果要查看 nat 表，可以加上 -t nat 参数。
-F：清楚规则链中已有的条目；
-Z：清空规则链中的数据包计算器和字节计数器；
-N：创建新的用户自定义规则链；
-P：定义规则链中的默认目标；
-h：显示帮助信息；
-p：指定要匹配的数据包协议类型；
-m 用于指定扩展模块的名称
-s：指定要匹配的数据包源ip地址；
-j<目标>：指定要跳转的目标；
-i<网络接口>：指定数据包进入本机的网络接口；
-o<网络接口>：指定数据包要离开本机所使用的网络接口。
-n 表示不对 IP 地址进行反查，加上这个参数显示速度将会加快。
-v 表示输出详细信息，包含通过该规则的数据包数量、总字节数以及相应的网络接口。
```

iptables 命令常用匹配参数及各自的功能：

```shell
参 数    功 能
[!]-p  匹配协议，! 表示取反
[!]-s  匹配源地址
[!]-d  匹配目标地址
[!]-i  匹配入站网卡接口
[!]-o  匹配出站网卡接口
[!]--sport  匹配源端口
[!]--dport  匹配目标端口
[!]--src-range  匹配源地址范围
[!]--dst-range  匹配目标地址范围
[!]--limit  四配数据表速率
[!]--mac-source  匹配源MAC地址
[!]--sports  匹配源端口
[!]--dports  匹配目标端口
[!]--stste  匹配状态（INVALID、ESTABLISHED、NEW、RELATED)
[!]--string  匹配应用层字串
```

iptables 命令选项输入顺序：

```shell
iptables -t 表名 <-A/I/D/R> 规则链名 [规则号] <-i/o 网卡名> -p 协议名 <-s 源IP/源子网> --sport 源端口 <-d 目标IP/目标子网> --dport 目标端口 -j 动作
```

**表名包括：**

```shell
raw：高级功能，如：网址过滤。
mangle：数据包修改（QOS），用于实现服务质量。
net：地址转换，用于网关路由器。
filter：包过滤，用于防火墙规则。
```

**规则链名包括：**

```shell
INPUT链：处理输入数据包。
OUTPUT链：处理输出数据包。
PORWARD链：处理转发数据包。
PREROUTING链：用于目标地址转换（DNAT）。
POSTOUTING链：用于源地址转换（SNAT）。
```

**动作包括：**

```shell
ACCEPT：接收数据包,允许数据包通过。
DROP：丢弃数据包。
REJECT  拒绝数据包通过
REDIRECT：重定向、映射、透明代理。
SNAT：源地址转换。
DNAT：目标地址转换。
MASQUERADE：IP伪装（NAT）,地址欺骗，用于ADSL。
LOG：将数据包信息记录 syslog 曰志。
```

## 实例

注意：有时需要删除的规则较长，删除时需要写一大串的代码，这样比较容易写错，这时可以先使用 --line-numbers 找出该条规则的行号，再通过行号删除规则。

**查看规则:**

```shell
iptables -nvL
iptables -nvL --line-numbers
```

解析：

```shell
-A DOCKER ! -i br-449595784b72 -p tcp -m tcp --dport 36381 -j DNAT --to-destination 192.168.176.2:6379
-A DOCKER ! -i br-83c273379ef8 -p tcp -m tcp --dport 5000 -j DNAT --to-destination 172.18.0.2:80
```

使用-p 选项指定了协议名称，使用扩展匹配条件 –dport 指定了目标端口，在使用扩展匹配条件的时候，如果没有使用-m 指定使用哪个扩展模块，iptables 会默认使用"-m 协议名"，而协议名就是-p 选项对应的协议名，上例中，-p 对应的值为 tcp，所以默认调用的扩展模块就为-m tcp，如果-p 对应的值为 udp，那么默认调用的扩展模块就为-m udp。

**清除已有 iptables 规则:**

```sh
iptables -F
iptables -X
iptables -Z
```

**添加规则（开放指定的端口）:**

添加规则有两个参数分别是 -A 和 -I。其中 -A 是添加到规则的末尾；-I 可以插入到指定位置，没有指定位置的话默认插入到规则的首部。

```sh
iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT               #允许本地回环接口(即运行本机访问本机)
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT    #允许已建立的或相关连的通行
iptables -A OUTPUT -j ACCEPT         #允许所有本机向外的访问
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    #允许访问22端口
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    #允许访问80端口
iptables -A INPUT -p tcp --dport 21 -j ACCEPT    #允许ftp服务的21端口
iptables -A INPUT -p tcp --dport 20 -j ACCEPT    #允许FTP服务的20端口
iptables -A INPUT -j reject       #禁止其他未允许的规则访问
iptables -A FORWARD -j REJECT     #禁止其他未允许的规则访问
```

**屏蔽 IP:**

用-I 往前追加，最先匹配。

屏蔽 docker:

```shell
iptables -I DOCKER -p tcp --dport 6379 -j DROP
iptables -I DOCKER -s 127.0.0.1 -p tcp --dport 6379 -j ACCEPT
iptables -L -n
```

```sh
iptables -I INPUT -s 123.45.6.7 -j DROP       #屏蔽单个IP的命令
iptables -I INPUT -s 123.0.0.0/8 -j DROP      #封整个段即从123.0.0.1到123.255.255.254的命令
iptables -I INPUT -s 124.45.0.0/16 -j DROP    #封IP段即从123.45.0.1到123.45.255.254的命令
iptables -I INPUT -s 123.45.6.0/24 -j DROP    #封IP段即从123.45.6.1到123.45.6.254的命令是
```

**修改规则:**

在修改规则时需要使用-R 参数。

```shell
iptables -R INPUT 6 -s 194.168.10.5 -j ACCEPT
iptables -nL --line-number
```

**查看已添加的 iptables 规则:**

```sh
iptables -L -n -v
```

**删除已添加的 iptables 规则:**

删除规则必须使用 -D 参数。

将所有 iptables 以序号标记显示，执行：

```sh
iptables -L -n --line-numbers
```

比如要删除 INPUT 里序号为 8 的规则，执行：

```sh
iptables -D INPUT 8
```

## 防火墙的备份与还原

默认的 iptables 防火墙规则会立刻生效，但如果不保存，当计算机重启后所有的规则都会丢失，所以对防火墙规则进行及时保存的操作是非常必要的。

iptables 软件包提供了两个非常有用的工具，我们可以使用这两个工具处理大量的防火墙规则。这两个工具分别是 iptables-save 和 iptables-restore，使用该工具可以实现防火墙规则的保存与还原。这两个工具的最大优势是处理庞大的规则集时速度非常快。

CentOS 7 系统中防火墙规则默认保存在 /etc/sysconfig/iptables 文件中，使用 iptables-save 将规则保存至该文件中可以实现保存防火墙规则的作用，计算机重启后会自动加载该文件中的规则。
如果使用 iptables-save 将规则保存至其他位置，可以实现备份防火墙规则的作用。
当防火墙规则需要做还原操作时，可以使用 iptables-restore 将备份文件直接导入当前防火墙规则。

```shell
iptables-save > /etc/sysconfig/iptables
# 或者用iptables-service保存【这个做法需要安装iptables-services，实际效果和上面那句一样】
service iptables save

# 然后在系统重启后手动加载此配置文件
iptables-restore < /etc/sysconfig/iptables

# 或着手动重启iptables【这个做法需要安装iptables-services】
systemctl restart iptables
```

### iptables-save 命令

iptables-save 命令用来批量导出 Linux 防火墙规则，语法介绍如下：

直接执行 iptables-save 命令：

```shell
# 保存在默认文件夹中（保存防火墙规则） (CentOS7)
iptables-save > /etc/sysconfig/iptables

# 保存在其他位置（备份防火墙规则）：
iptables-save > 文件名称
```

显示出当前启用的所有规则，按照 raw、mangle、nat、filter 表的顺序依次列出。

### iptables-restore 命令

iptables-restore 命令可以批量导入 Linux 防火墙规则，同时也需要结合重定向输入来指定备份文件的位置。

命令如下：

```shell
iptables-restore < 文件名称
```

注意，导入的文件必须是使用 iptables-save 工具导出来的才可以。

先使用 iptables-restore 命令还原 text 文件，然后使用 iptables -t nat -nvL 命令查看清空的规则是否已经还原。

参考：

[Linux iptables 命令详解](https://zhuanlan.zhihu.com/p/513470164)

[iptables 命令](https://man.linuxde.net/iptables)
