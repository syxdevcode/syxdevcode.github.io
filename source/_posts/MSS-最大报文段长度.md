---
title: MSS-最大报文段长度
date: 2021-02-28 17:40:07
tags:
- TCP协议
- 计算机基础
- 网络基础
- IP网络
- Ethernet
categories:
- Ethernet
---

最大报文段长度（MSS）是TCP协议的一个选项，用于在TCP连接建立时，收发双方协商通信时每一个报文段所能承载的最大数据长度（不包括文段头）。

## 开放式互联网模型

开放式系统互联模型（OpenSystemInterconnection Model，简称为OSI模型）是一种互联网概念化模型，由国际标准化组织(InternationalOrganization forStandardization，简称为ISO)提出，定义于ISO/IEC 7498-1。OSI模型将互联网分为七层，由最高层（用户端）到最底层（物理层面）排列为：第7层 应用层（Application Layer）;第6层 表达层（Presentation Layer）；第5层 会话层（Session Layer）；第4层 传输层（Transport Layer）；第3层 网络层（Network Layer）；第2层 数据链接层（Data Link Layer）第1层 物理层（Physical Layer）；本词条MSS是第四层传输层中的一种协议（TCP）的选项之一。

![503d269759ee3d6d55fbf8fa925e7a224f4a20a470e3.png](/img/503d269759ee3d6d55fbf8fa925e7a224f4a20a470e3.png)

## 区分MSS与MTU

最大报文段长度（MSS）与最大传输单元（Maximum Transmission Unit, MTU）均是协议用来定义最大长度的。不同的是，MTU应用于OSI模型的第二层数据链接层，并无具体针对的协议。MTU限制了数据链接层上可以传输的数据包的大小，也因此限制了上层（网络层）的数据包大小。例如，如果已知某局域网的MTU为1500字节，则在网络层的因特网协议（Internet Protocol, IP）里，最大的数据包大小为1500字节（包含IP协议头）。MSS针对的是OSI模型里第四层传输层的TCP协议。因为MSS应用的协议在数据链接层的上层，MSS会受到MTU的限制。

![d009b3de9c82d158ccbfd28051420ed8bc3eb135e4e3.png](/img/d009b3de9c82d158ccbfd28051420ed8bc3eb135e4e3.png)