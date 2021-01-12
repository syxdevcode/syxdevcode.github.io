---
title: Linux TcpDump工具
date: 2021-01-08 17:40:49
tags:
- Linux
- CentOS7
- 网络抓包
- TcpDump
- Linux优化
categories:
- TcpDump
---

## 简介

TcpDump 使用了独立于系统的 libpcap 的接口。libpcap 是linux平台下的网络数据包捕获函数包，大多数网络监控软件都以它为基础。tcpdump 调用libpcap 的接口在 linux 系统链路层抓包。而linux本身指定的许多访问控制规则都是基于三层或三层以上的过滤规则，所以 tcpdump 可以抓取过滤规则之前的数据包。使用tcpdump 需要有root权限。

## 网卡基础知识

### 网卡的不同接受模式

* 广播模式：该模式下网卡能接收网络中的广播信息。
* 组播模式：该模式下网卡能接收网络中的组播信息。
* 直接模式：该模式下网卡能接收网络中目的地址为自己的数据包。
* 混杂模式：该模式下网卡能接收网络中一切通过该网卡的数据包。

注：网卡混杂模式：是网卡的一种工作模式，一般在抓取网卡数据包时使用。

```sh
device eth0 entered promiscuous mode # 是指网卡 eth0 进入了混杂模式。
device eth0 left promiscuous mode # 网卡 eth0 离开了混杂模式。
ifconfig eth0 promisc # 设置网卡eth0为混杂模式
ifconfig eth0 -promisc # 取消网卡eth0的混杂模式
```

### 网络中数据包的分类

* 广播包：指IP子网内广播的数据包。适用范围较小只在本地子网有效。通过路由器和网络设备控制传输。
* 单播包：发送者和每一接受者中点对点的网络连接。
* 组播包：借助组播路由协议建立树形路由，在尽可能远的分岔路口才开始复制和奋发。（224.0.0.0~224.0.0.255是预留的组播地址）

### 数据包接收流程

* 网卡收到数据包，获取数据包中的目的MAC地址。
* 根据网卡驱动设置的网卡接受模式去判断是否接受该数据。
* 若接受该数据：发出中断信号通知CPU；CPU收到中断信号后根据发出该中断信号的网卡驱动程序的网卡驱动程序地址调用网卡驱动程序；网卡驱动程序处理数据；驱动程序将数据放入信号堆栈；系统接触到数据。
* 若不接受该数据：网卡直接丢弃该数据；系统不会接触到数据。

## 安装

```sh
# 解锁
# 如果不解锁，可能造成错误：Couldn't find user 'tcpdump'
chattr -i /etc/passwd /etc/shadow /etc/group /etc/gshadow

# 安装
yum install tcpdump
```

## tcpdump命令格式

```sh
tcpdump [ -DenNqvX ] [ -c count ] [ -F file ] [ -i interface ] [ -r file ]
        [ -s snaplen ] [ -w file ] [ expression ]
```

* -a 尝试将网络和广播地址转换成名称。
* -c<数据包数目> 收到指定的数据包数目后，就停止进行倾倒操作。
* -d 把编译过的数据包编码转换成可阅读的格式，并倾倒到标准输出。
* -dd 把编译过的数据包编码转换成C语言的格式，并倾倒到标准输出。
* -ddd 把编译过的数据包编码转换成十进制数字的格式，并倾倒到标准输出。
* -e 在每列倾倒资料上显示连接层级的文件头。
* -f 用数字显示网际网络地址。
* -F<表达文件> 指定内含表达方式的文件。
* -i<网络界面> 使用指定的网络截面送出数据包。
* -l 使用标准输出列的缓冲区。
* -n 不把主机的网络地址转换成名字。
* -N 不列出域名。
* -O 不将数据包编码最佳化。
* -p 不让网络界面进入混杂模式。
* -q 快速输出，仅列出少数的传输协议信息。
* -r<数据包文件> 从指定的文件读取数据包数据。
* -s<数据包大小> 设置每个数据包的大小。
* -S 用绝对而非相对数值列出TCP关联数。
* -t 在每列倾倒资料上不显示时间戳记。
* -tt 在每列倾倒资料上显示未经格式化的时间戳记。
* -T<数据包类型> 强制将表达方式所指定的数据包转译成设置的数据包类型。
* -v 详细显示指令执行过程。
* -vv 更详细显示指令执行过程。
* -vvv：产生比-vv更详细的输出。
* -x 用十六进制字码列出数据包资料。
* -w<数据包文件> 把数据包数据写入指定的文件。

## 常用实例

```sh
# 监听特定主机
tcpdump host 182.254.38.55

# 特定来源
tcpdump src host hostname

# 特定目标地址
tcpdump dst host hostname

# 特定端口
tcpdump port 3000

# 监听TCP/UDP
tcpdump tcp

# 来源主机+端口+TCP
# 监听来自主机 10.207.22.254 在端口 8011 上的TCP数据包
tcpdump tcp port 8011 and src host 10.207.22.254

# 监听特定主机之间的通信
tcpdump ip host 210.27.48.1 and 210.27.48.2

# 保存到本地
# 备注：tcpdump 默认会将输出写到缓冲区，只有缓冲区内容达到一定的大小，或者tcpdump退出时，才会将输出写到本地磁盘
tcpdump -n -vvv -c 1000 -w /ldjc/data.pcap
```

## 输出

TCP协议行的典型格式如下：

[Timestamp] [Protocol] [Src IP].[Src Port] > [Dst IP].[Dst Port]: [Flags], [Seq], [Ack], [Win Size], [Options], [Data Length]

让我们逐个字段进行说明，并解释以下内容：

```sh
21:53:20.460144 IP 192.168.182.166.57494 > 35.222.85.5.80: Flags [P.], seq 1:88, ack 1, win 29200,  
options [nop,nop,TS val 1067794587 ecr 2600218930], length 87
```

```sh
21:53:20.460144 - 捕获的数据包的时间戳为本地时间，并使用以下格式：hours：minutes：seconds.frac，
        其中frac是自午夜以来的几分之一秒。
IP - 分组协议。 在这种情况下，IP表示Internet协议版本4（IPv4）。
192.168.182.166.57494 - 源IP地址和端口，以点（.）分隔。
35.222.85.5.80 - 目的IP地址和端口，以点号（.）分隔。
```

TCP 标志字段。 在此示例中，[P.] 表示推送确认数据包，用于确认前一个数据包并发送数据。 其他典型标志字段值如下：

```sh
[.] - ACK (Acknowledgment)
[S] - SYN (Start Connection)
[P] - PSH (Push Data)
[F] - FIN (Finish Connection)
[R] - RST (Reset Connection)
[S.] - SYN-ACK (SynAcK Packet)
```

```sh
seq 1:88 - 序列号在first：last表示法中。 它显示了数据包中包含的数据数量。 
        除了数据流中的第一个数据包（其中这些数字是绝对的）以外，所有后续数据包均用作相对字节位置。 
        在此示例中，数字为1:88，表示此数据包包含数据流的字节1至88。 使用-S选项可打印绝对序列号。
ack 1 - 确认号（acknowledgment number）是此连接另一端所期望的下一个数据的序列号。
win 29200 - 窗口号是接收缓冲区中可用字节的数目。
options [nop,nop,TS val 1067794587 ecr 2600218930] - TCP选项。 
        使用 nop 或 “ no operation” 填充使TCP报头为4字节的倍数。 
        TS val 是TCP时间戳，而 ecr 表示回显应答。 请访问 IANA 文档以获取有关 TCP 选项的更多信息。
length 87 - 有效载荷数据的长度
```

参考：

[tcpdump抓包wireshark分析](https://blog.csdn.net/godop/article/details/80986966)

[一文搞定tcpdump基本用法](https://blog.csdn.net/chinaltx/article/details/87469933)

[Linux下抓包命令tcpdump详解](https://www.linuxidc.com/Linux/2020-02/162226.htm)

[Linux基础：用tcpdump抓包](https://www.cnblogs.com/chyingp/p/linux-command-tcpdump.html)

[聊聊 tcpdump 与 Wireshark 抓包分析](https://www.jianshu.com/p/8d9accf1d2f1)