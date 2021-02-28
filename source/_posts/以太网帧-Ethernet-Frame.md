---
title: 以太网帧-Ethernet Frame
date: 2021-01-29 17:10:06
tags:
- TCP协议
- 计算机基础
- 网络基础
- IP网络
- Ethernet
categories:
- Ethernet
---

![ethernet-frame.gif](/img/ethernet-frame.gif)

octets：（Bytes）字节
Frame：帧

![d009b3de9c82d158ccbfd28051420ed8bc3eb135e4e3.png](/img/d009b3de9c82d158ccbfd28051420ed8bc3eb135e4e3.png)

## Ethernet帧格式历史

1980 DEC,Intel,Xerox 制订了 Ethernet I 的标准

1982 DEC,Intel,Xerox 又制订了 Ehternet II 的标准

1982 IEEE 开始研究 Ethernet 的国际标准 802.3

1983 迫不及待的 Novell 基于 IEEE 802.3 的原始版开发了专用的 Ethernet 帧格式 (因此 802.3 Raw 先于 IEEE 802.3 出台.)

1985 IEEE 推出 IEEE 802.3 规范,

后来为解决 Ethernet II 与 802.3 帧格式 的兼容问题,推出折衷的 Ethernet SNAP 格式

(其中早期的 Ethernet I 已经完全被其他帧格式取代了 ,所以现在 Ethernet 只能见到后面几种 Ethernet的帧格式,现在大部分的网络设备都支持这几种 Ethernet 的帧格式,如:cisco 的路由器再设定 Ethernet 接口时可以指定不同的以太网的帧格式:arpa,sap,snap,novell-ether)

今天的实际环境中大多数TCP/IP设备都使用Ethernet V2格式的帧。
这是因为第一种大规模使用的TCP/IP系统(4.2/3 BSD UNIX)的出现时间介于RFC 894和RFC 1042之间，
它为了避免不能和别的主机互操作的风险而采用了RFC 894的实现；
也由于大家都抱着这种想法，所以 802.3 标准并没有如预期那样得到普及；
 
CISCO设备的Ethernet Interface默认封装格式是ARPA(Ethernet V2)

**不同厂商对这几种帧格式不同的叫法**

| Frame Type | Novell公司 | Cisco 公司 |
| :-----| :---- | :---- |
| Ethernet Version 2    |  Ethernet_II       |  arpa
| 802.3 Raw             |  Ethernet_802.3    |  novell_ether
| IEEE 802.3/802.2      |  Ethernet_802.2    |  sap
| IEEE 802.3/802.2 SNAP |  ETHERNET_SNAP     |  snap

## 帧格式

### Ethernet I (V1)

这是最原始的一种格式，是由 Xerox PARC 提出的 3Mbps CSMA/CD 以太网标准的封装格式，
后来在1980年由 DEC，Intel 和 Xerox 标准化形成 Ethernet V1 标准；

### Ethernet II (V2) (ARPA)

这是最常见的一种以太网帧格式，也是今天以太网的事实标准，由DEC，Intel 和Xerox [简称 DIX以太网联盟] 在1982年公布其标准，主要更改了Ethernet V1 的电气特性和物理接口， 在帧格式上并无变化；Ethernet V2 出现后迅速取代 Ethernet V1 成为以太网事实标准；

Ethernet V2 帧头（Frame Header）结构为： 6bytes的源地址（源MAC） + 6bytes的目标地址（目标MAC） + 2Bytes协议类型字段（用于标示封装在这个 Frame 里面数据的类型)，
数据长度 46--1500 Bytes，4Bytes的帧校验。

Ethernet V2 类型以太网帧的最小长度为64字节（6＋6＋2＋46＋4），最大长度为1518字节（6＋6＋2＋1500＋4）。

常见协议类型如下：

0x0800 　　IP协议数据，
0x86dd 　　IPv6协议数据，
0x809B 　　AppleTalk协议数据，
0x8138 　　Novell类型协议数据等。
0x0806 　　ARP
0x0600 　　XNS (Xerox)
0x6003 　　DECNET

如果协议类型字段取值为0000-05dc(十进制的0-1500)，则该帧就不是 Ethernet V2(ARPA) 类型了，而是下面的三种 802.3帧 类型之一；

Ethernet V2 可以支持 TCP/IP，Novell IPX/SPX，Apple Talk Phase I 等协议；RFC 894 定义了IP报文在Ethernet V2 上的封装格式；

### Novell Ethernet （802.3 Raw） novell_ether

这是1983年 Novell 发布其划时代的 Netware/86 网络套件时，采用的私有以太网帧格式，该格式以当时尚未正式发布的 802.3 标准为基础；

但是当两年以后 IEEE正式发布 802.3 标准时情况发生了变化 — IEEE 在802.3帧头中又加入了802.2 LLC(Logical Link Control)头，这使得 Novell 的 802.3 RAW 格式跟正式的 IEEE 802.3 标准互不兼容；可以看到在 Novell 的 RAW 802.3 帧结构中并没有标志协议类型的字段，而只有 Length 字段(2bytes,取值为0000-05dc，即十进制的0-1500)，因为 RAW 802.3帧 只支持 IPX/SPX 一种协议；

802.3 Raw 帧头与 Ethernet 帧头有所不同，其中 Ethernet II 帧头中的类型域变成了长度域，后面接着的两个字节为 0xFFFF,用于标示这个帧是 Novell Ether 类型的 Frame,由于前面的 0xFFFF 站掉了两个字节所以数据域缩小为 44-1498 个字节,帧校验不变。

![802.3Raw.png](/img/802.3_Raw.png)

* 目标地址：此数据包的目标MAC地址。
* 源地址：此数据包的源MAC地址。
* 长度：帧包含的数量必须或等于 1500。(在 Ethernet 802.3 raw 类型以太网帧中，原来 Ethernet II 类型以太网帧中的类型字段被"总长度"字段所取代，它指明其后数据域的长度，其取值范围为：46-1500。)
* 数据：高层协议（IPX/SPX）、数据和填充符，范围在46～1500字节。(长度紧跟着的接下来的2个字节是固定不变的16进制数 0xFFFF，它标识此帧为 Novell 以太类型数据帧。)
* FCS：数据帧校验序列，用于确定数据包在传输过程中是否损坏。

### IEEE 802.3/802.2 LLC (Logical Link Control),sap

这是IEEE 正式的 802.3 标准，它由 Ethernet V2 发展而来。

IEEE 802.3 的 Frame Header 和 Ethernet II 的帧头有所不同,IEEE 802.3 将 Ethernet V2 帧头的协议类型字段替换为帧长度字段(取值为0000-05dc;十进制的1500)。

其中又引入 802.2 协议(LLC,Logical Link Control) 在 802.3 帧头后面添加了一个 LLC 首部,LLC头包含目的 服务访问点(DSAP,Destination Service Access Point)、源服务访问点（SSAP,Source Service Access Point）和 控制（Control）字段。

IEEE 802.3 把 DLC (数据链路控制，Digital Loop Carrier) 层分隔成明显的两个子层：MAC 层和 LLC 层，其中 MAC 层主要是指示硬件目的地址和源地址，LLC层用来提供一些服务：

* 通过SAP地址来辨别接收和发送方法
* 兼容无连接和面向连接服务
* 提供子网访问协议（Sub-network Access Protocol，SNAP），类型字段即由它的首部给出。

MAC层要保证最小帧长度不小于64字节，如果数据不满足64字节长度就必须进行填充。

利用 Sniffer 等协议分析工具去捕获的 IEEE 802.3 帧的解码，可以看到在 DLC 层源地址后紧跟着就是 802.3 的长度（Length）字段 0026，它小于 05FF（二进制1535），可以肯定它不是 Ethernet V2 的帧，而接下来的 Offset 0E 处的值 4242 （代表DSAP和SSAP），既不是 Novell 802.3 Raw 的特征值 FFFF ，也不是 IEEE 802.3 SNAP 的特征值 AAAA ，因此它肯定是一个 IEEE 802.3 的帧。

以太网类型码：

![ETHERNET_type_code.png](/img/ETHERNET_type_code.png)

decimal：十进制
octal：八进制

**802.2 SAP (Service Access Point) 介绍**

为了区别 802.3 数据帧中所封装的数据类型，IEEE 引入了 802.2 SAP 和 SNAP 的标准。它们工作在数据链路层的 LLC（逻辑链路控制）子层。

通过在802.3帧的数据字段中划分出被称为服务访问点（SAP）的新区域来解决识别上层协议的问题，这就是 802.2 SAP。

**LLC标准 介绍**

LLC 标准包括两个服务访问点，源服务访问点（SSAP）和目标服务访问点（DSAP）。每个SAP只有1字节长，而其中仅保留了6比特用于标识上层协议，所能标识的协议数有限。

因此，又开发出另外一种解决方案，在 802.2 SAP 的基础上又新添加了一个 2字节长的类型域（同时将SAP的值置为AA），使其可以标识更多的上层协议类型，这就是 802.2 SNAP。

常见SAP值：
* 0：Null LSAP[IEEE] 
* 4：SNA Path Control[IEEE] 
* 6：DOD IP[79,JBP] 
* AA：SNAP[IEEE] 
* FE：ISO DIS 8473[52,JXJ] 
* FF：Global DSAP[IEEE]

![IEEE_802.3.png](/img/IEEE_802.3.png)

* 目标地址：此数据包的目标mac地址；
* 源地址：此数据包的源mac地址；
* 长度：帧包含的数据量必须小于或等于1500（16进制的05DC）；
* DSAP：目标服务存取点（Destination  Service Access Point）；
* SSAP：源服务存取点（Source Service Access  Point）；
* 控制：无连接或面向连接的C；
* 数据：高层协议、数据和填充符;
* FCS：数据帧校验序列，用于确定数据包在传输过程中是否损坏。

### Ethernet SNAP (IEEE 802.3/802.2 SNAP, ETHERNET_SNAP, snap)

SNAP (Sub-Network Access Protocol)子网访问协议，是逻辑链路控制（Logical Link Control）的一个子集，它允许协议不用通过 服务访问点（SAP）即可实现 IEEE 兼容的 MAC 层功能，因此它在 DSAP 和 SSAP域里的值是固定的 （AAAA）。也正源于此，它需要额外提供5个字节的头来指定接收方法，3个字节标识厂商代码，2个字节标识上层协议。

其 MAC 层保证数据帧长度不小于64字节，不足的话需要进行数据填充。

Sniffer 捕获的 IEEE 802.3 SNAP 帧的解码，可以看到在 DLC 层源地址后紧跟着就是 802.3 的长度（Length）字段 0175，它小于 05FF，可以肯定它不是 Ethernet V2 的帧，而接下来的 Offset 0E 处的值 AAAA （代表DSAP和SSAP），这是 IEEE 802.3 SNAP 的特征值 AAAA ，因此可以断定它是一个 IEEE 802.3 SNAP 的帧。

SNAP Frame 与 802.3/802.2 Frame 的最大区别是增加了一个 5 Bytes 的 SNAP ID，其中前面3个byte通常与源mac地址的前三个bytes相同为厂商代码，有时也可设为0,后2 bytes 与 Ethernet II 的类型域相同。

![ETHERNET_SNAP.png](/img/ETHERNET_SNAP.png)

## 小结

| Frame Type | Header & CRC | Data Min |Data Max|
| :-----| :---- | :---- | :--- |
| Ethernet II (DIX)	|  18 | 46 | 1500
| 802.3 (IEEE)	    |  21 | 43 | 1497
| SNAP	            |  26 | 38 | 1492

* Ethernet II 和 IEEE 802.3 是局域网里最常见的帧
* Ethernet II 可以装载的数据长度是46---1500;  
* IEEE802.3 SAP 可以装装的数据长度是43---1497; 
* IEEE 802.3 SNAP 可以装载的数据长度是38---1492；
* Ethernet II 不提供 MAC 层的数据填充功能;
* IEEE802.3 SAP 和 SNAP 都提供数据填充功能.

**Ethernet V2 帧与IEEE 802.3 帧的比较**

Ethernet V2 可以装载的最大数据长度是1500字节，而 IEEE 802.3 可以装载的最大数据是1492字节（SNAP）或是1497字节; Ethernet V2 不提供 MAC层 的数据填充功能，而 IEEE 802.3 不仅提供该功能，还具备服务访问点（SAP）和 SNAP 层，能够提供更有效的数据链路层控制和更好的传输保证。那么我们可以得出这样的结 论：Ethernet V2 比 IEEE 802.3 更适合于传输大量的数据，但 Ethernet V2 缺乏数据链路层的控制，不利于传输需要严格传输控制的数据，这也正是 IEEE 802.3 的优势所在，越需要严格传输控制的应用，越需要用 IEEE 802.3或 SNAP 来封装，但 IEEE 802.3 也不可避免的带来数据装载量的损失，因此该格式的封装往往用在较少数据量承载但又需要严格控制 传输的应用中。

在实际应用中，我们会发现，大多数应用的以太网数据包是 Ethernet V2 的帧（如HTTP、FTP、SMTP、POP3等应用），而交换机之间的BPDU（桥协议数据单元）数据包则是IEEE 802.3的帧，VLAN Trunk 协议如 802.1Q 和Cisco的CDP（思科发现协议）等则是采用 IEEE 802.3 SNAP 的帧。可以利用Sniffer等协议分析工具去捕捉数据包。

参考：

[以太网帧与ieee 802.3帧](https://blog.csdn.net/guoshaobei/article/details/4768514)

[以太网类型码(Ethernet type codes)](https://blog.csdn.net/yshe_xun/article/details/7636078)

[Ethernet frame](https://www.cnblogs.com/xlmeng1988/articles/2445619.html)

[ETHERNET帧结构](https://www.cnblogs.com/mylinux/p/5553242.html)

[车小胖谈网络：Ethernet Frame](https://zhuanlan.zhihu.com/p/21318925)


