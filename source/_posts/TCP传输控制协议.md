---
title: TCP传输控制协议
date: 2021-01-11 15:05:11
tags:
- TCP协议
- 计算机基础
categories:
- TCP协议
---

## TCP 报文格式简介

* TCP 报文由 `TCP Header` 和 `TCP` 数据组成。
* **TCP Header 的最大长度为 60 字节(byte)**，而**必须要有的固定长度**也就是图一的前5层的**20字节(byte)**，每层占有 `32bit`，也就是 `32/8=4` 字节，5层，`5*4 = 20` 字节，那么第六层的可选项和填充也就是**Tcp Options字段最大为60-20=40字节(byte)**。填充是为了使TCP首部为4字节（32bit）的整数倍。

## TCP首部格式

![tcp2.png](/img/tcp2.png)

**Source Port**：源端口，16位(bit)，2个字节(byte)。
**Destination Port**：目的端口，16位，2个字节。
**Sequence Number**：序号，发送数据包中的第一个字节的序列号，32位。
**Acknowledgment Number**：确认序列号，32位。
**Data Offset**：数据偏移，4位，该字段的值是TCP首部（包括选项）长度除以4。
**标志位**：6位，URG 表示 `Urgent Pointer` 字段有意义：
`ACK` 表示 `Acknowledgment Number` 字段有意义
`PSH` 表示 `Push` 功能，RST 表示复位 TCP 连接
`SYN` 表示 `SYN` 报文（在建立 TCP 连接的时候使用）
`FIN` 表示没有数据需要发送了（在关闭 TCP 连接的时候使用）
**Window**：窗口，表示接收缓冲区的空闲空间，16位，2个字节，用来告诉TCP连接对端自己能够接收的最大数据长度。
**Checksum**：校验和，16位，2个字节。
**Urgent Pointers**：紧急指针，16位,2个字节，只有 URG 标志位被设置时该字段才有意义，表示紧急数据相对序列号（Sequence Number字段的值）的偏移。
**选项和填充**：最常见的可选字段是最长报文大小，又称为 MSS（Maximum Segment Size），每个连接方通常都在通信的第一个报文段（为建立连接而设置SYN标志为1的那个段）中指明这个选项，它表示本端所能接受的最大报文段的长度。选项长度不一定是32位的整数倍，所以要加填充位，即在这个字段中加入额外的零，以保证TCP头是32的整数倍。
**数据**：TCP 报文段中的数据部分是可选的。在一个连接建立和一个连接终止时，双方交换的报文段仅有 TCP 首部。如果一方没有数据要发送，也使用没有任何数据的首部来确认收到的数据。在处理超时的许多情况中，也会发送不带任何数据的报文段。

## TCP Options字段

Tcp Options 字段的最大长度为40字节。Tcp Options 字段的一般数据结构如图所示：

* Kind(1字节)
* Length(1字节)
* Info(n字节)

选项的第一个字段 kind 说明选项的类型。有的 TCP 选项没有后面两个字段，仅包含1字节的kind字段。第二个字段length（如果有的话）指定该选项的总长度，该长度包括kind字段和length字段占据的2字节。第三个字段 info（如果有的话）是选项的具体信息。

![tcpoptions.png](/img/tcpoptions.png)

* 第一个kind= 2，表示最大报文段长度（Max Segment Size，MSS），TCP 模块通常将 MSS 设置为（MTU-40, MTU[Maximum Transmission Unit，缩写 MTU，中文名是：最大传输单元]）字节（减掉的这40字节包括20字节的TCP头部和20字节的IP头部）。这样携带 TCP 报文段的IP数据报的长度就不会超过 MTU（假设TCP头部和IP头部都不包含选项字段，并且这也是一般情况），从而避免本机发生IP分片。对以太网而言，MSS 值是1460（1500-40）字节。
* kind= 4，表示支持 SACK，SACK Block（收到乱序数据）。
* kind = 8，代表 Timestamps，即时间戳，启用 Timestamp Option 后，每个 TCP Segment 中都会带有 Timestamp Option，其中包含了两个 32bit 的 Timestamp 也就是各四个字节的 Timestamp Value（TSval）和 Timestamp Echo Reply（TSecr）。发送方在发送报文段时把当前时钟的时间值放入时间戳字段，接收方在确认该报文段时把时间戳字段值复制到时间戳回送回答字段。因此，发送方在收到确认报文后，可以准确计算出RTT。

## 工作方式

## 建立连接

![tcp1](/img/tcp1.gif)

TCP是因特网中的传输层协议，使用三次握手协议建立连接。当主动方发出 `SYN` 连接请求后，等待对方回答 `SYN+ACK`，并最终对对方的 `SYN` 执行 `ACK` 确认。

TCP三次握手的过程如下：

* 客户端发送 `SYN（SEQ=x）` 报文给服务器端，进入 `SYN_SEND` 状态。
* 服务器端收到 `SYN` 报文，回应一个 `SYN （SEQ=y）ACK（ACK=x+1）` 报文，进入 `SYN_RECV` 状态。
* 客户端收到服务器端的 `SYN` 报文，回应一个 `ACK（ACK=y+1）` 报文，进入 `Established(已获确认的)` 状态。

三次握手完成，TCP客户端和服务器端成功地建立连接，可以开始传输数据了。

### 连接终止

![tcp3.gif](/img/tcp3.gif)

建立一个连接需要三次握手，而终止一个连接要经过四次握手，这是由TCP的 `半关闭`（`half-close`）造成的。

* （1）某个应用进程首先调用 `close`，称该端执行 `主动关闭（active close）`。该端的TCP于是发送一个 `FIN` 分节，表示数据发送完毕。
* （2） 接收到这个 `FIN` 的对端执行 `被动关闭（passive close）`，这个 `FIN` 由TCP确认。

注意：`FIN` 的接收也作为一个文件结束符（`end-of-file`）传递给接收端应用进程，放在已排队等候该应用进程接收的任何其他数据之后，因为，`FIN` 的接收意味着接收端应用进程在相应连接上再无额外数据可接收。
* （3） 一段时间后，接收到这个文件结束符的应用进程将调用 `close` 关闭它的套接字。这导致它的 `TCP` 也发送一个 `FIN`。
* （4） 接收这个最终 `FIN` 的原发送端TCP（即执行主动关闭的那一端）确认这个 `FIN`。

既然每个方向都需要一个`FIN`和一个 `ACK` ，因此通常需要4个分节。

注意：
（1） `通常` 是指，某些情况下，步骤1的 `FIN` 随数据一起发送，另外，步骤2和步骤3发送的分节都出自执行被动关闭那一端，有可能被合并成一个分节。
（2） 在 步骤2 与 步骤3 之间，从执行 被动关闭一端 到 执行主动关闭一端 流动数据是可能的，这称为 `半关闭`（half-close）。
（3） 当一个Unix进程无论自愿地（调用exit或从main函数返回）还是非自愿地（收到一个终止本进程的信号）终止时，所有打开的描述符都被关闭，这也导致仍然打开的任何TCP连接上也发出一个`FIN`。
无论是客户还是服务器，任何一端都可以执行主动关闭。通常情况是，客户执行主动关闭，但是某些协议，例如，HTTP/1.0却由服务器执行主动关闭。

## TCP重连

![tcp重连.png](/img/tcp重连.png)

四元组：源IP地址、目的IP地址、源端口、目的端口
五元组：源IP地址、目的IP地址、协议号、源端口、目的端口
七元组：源IP地址、目的IP地址、协议号、源端口、目的端口、服务类型、接口索引

参考：

[Tcp报文简介以及头部选项字段(Tcp Options字段)](https://blog.csdn.net/Hollake/article/details/89327474)

[常用的TCP Option](https://blog.csdn.net/blakegao/article/details/19419237)

[浅析TCP中时间戳选项timestamp](https://blog.csdn.net/mary19920410/article/details/77255967)