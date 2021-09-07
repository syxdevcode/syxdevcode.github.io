---
title: DNS-RecordType记录类型
date: 2021-09-07 11:15:30
tags:
- 网络基础
- DNS
- 网络协议
categories:
- DNS
---

## A

A记录： 将域名指向一个IPv4地址（例如：100.100.100.100），需要增加A记录。

```s
example.com     A       12.34.56.78
hello.example.com       A       12.34.56.78
```

可以将不同的子域指向不同的 IP 地址。如果要将 example.com 的每个子域都指向同一个 IP，可以使用星号（`*`）作为子域：

```s
*.example.com   A       12.34.56.78
```

## AAAA

一个 AAAA 记录和 A 记录相似，不过是用于 IPv6 的 IP 地址。一个典型的 AAAA 记录如下所示：

```s
example.com     AAAA        0123:4567:89ab:cdef:0123:4567:89ab:cdef
```

## NS

NS记录： 域名解析服务器记录（`Name sever record`），是为域或子域设置对应的域名服务器。
如果要将子域名指定某个域名服务器来解析，需要设置NS记录。

域名的主要域名服务器记录既可以在注册商处设置，也可以在自己的区域文件中设置。一份典型的域名服务器记录（至少需要两个记录）如下所示：NS 记录的顺序无关紧要。DNS 请求随机发送到不同的服务器，如果一个主机无法响应，将查询另外一个主机。

```s
example.com     NS      ns1.linode.com.
example.com     NS      ns2.linode.com.
example.com     NS      ns3.linode.com.
example.com     NS      ns4.linode.com.
example.com     NS      ns5.linode.com.
```

## CNAME

CNAME 记录：称为规范名称记录（`Canonical Name record`），它将一个域或子域匹配到其它不同的域。
如果将域名指向一个域名，实现与被指向域名相同的访问效果，需要增加`CNAME`记录。
通过 CNAME 记录，DNS 查找则采用目标域的 DNS 解析作为别名的解析。下面举例：

```s
alias.com       CNAME   example.com.
example.com     A       12.34.56.78
```

## SOA

SOA记录： 起始权限记录（Start of Authority record），起始授权机构记录，NS用于标识多台域名解析服务器，SOA记录用于在众多NS记录中哪一台是主服务器。

SOA 记录中提到的单个域名服务器被视为动态 DNS 的第一服务器，并且是将区域文件传播到所有其他域名服务器之前，区域文件就要被更改完成的所在。

 SOA 记录：

```s
@   IN  SOA ns1.linode.com. admin.example.com. 2013062147 14400 14400 1209600 86400
```

注意：管理电子邮件地址使用句点（`.`）而不是 `@` 符号编写。

含义：

* **序列号**：此域的区域文件的修订号。它在文件更新时会发生变化。
* **刷新时间**：辅助 DNS 服务器在检查更改之前保留区域文件的时间（以秒为单位）。
* **重试时间**：辅助 DNS 服务器在重试传输失败的区域文件之前需要等待的时间。
* **过期时间**：辅助 DNS 服务器无法自行更新时，当前区域文件副本失效之前所等待的时间。
* **最短 TTL**：其他服务器应从该区域文件中保留缓存数据的最短时间。

## WKS

WKS：众所周知的业务描述

## CAA

DNS 证书颁发机构授权（CAA，Certification Authority Authorization）使用 DNS 来允许域的持有者指定 “哪些证书颁发机构能够为该域发放证书”。

## AXFR

AXFR 记录是一种用于 DNS 复制的记录（虽然当下有更先进的 DNS 复制方法）。AXFR 记录不是用于普通区域文件的。相反，它们应用于从 DNS 服务器，作用是从主 DNS 服务器上复制区域文件。

## PTR

PTR记录： PTR记录是A记录的逆向记录，又称做IP反查记录或指针记录，负责将IP反向解析为域名。

PTR 记录或称指针记录（`Pointer record`）将 IP 地址匹配至一个域或者子域，它允许反向的 DNS 查询工作。它执行的服务于 A 记录截然相反，因为它允许您查找与特定 IP 地址相关联的域。

添加 PTR 记录，需要创建一个有效且实时的 A 或 AAAA 记录，将所需的域指向该 IP。如果需要 IPv4 PTR 记录，请将域或子域指向IPv4 地址。如果需要 IPv6 PTR 记录，则将域指向IPv6 地址。除此之外，IPv4 和 IPv6 PTR 记录的工作方式是相同的。

## MX

MX记录：邮件交换记录（Mail exchanger record），建立电子邮箱服务，将指向邮件服务器地址，需要设置MX记录。建立邮箱时，一般会根据邮箱服务商提供的MX记录填写此记录。

```s
example.com         MX      10  mail.example.com.
mail.example.com    A           12.34.56.78
```

以上记录将`example.com`的邮件直接发送到`mail.example.com`这一服务器。目标域（上述的mail.example.com）需要有自己的 A 记录。理想情况下，MX 记录应指向同为其服务器主机名的域。

优先级是 MX 记录的另一个组成部分。这是记录类型和目标服务器之间写入的数字（在上例中为 10）。优先级允许您为特定域的邮件指定一个回退服务器（或多个服务器）。较低的数字代表较高的优先级。

下面是具有两个回退邮件服务器的域的示例：

```s
example.com         MX      10  mail_1.example.com
example.com         MX      20  mail_2.example.com
example.com         MX      30  mail_3.example.com
```

在此示例中，如果 `mail_1.example.com` 已关闭，则将传递邮件到 `mail_2.example.com`。如果`mail_2.example.com` 也是关闭，邮件将被发送到 `mail_3.example.com`。

## TXT

可任意填写，可为空。一般做一些验证记录时会使用此项，如：做SPF（反垃圾邮件）记录。

TXT记录或称文本记录（Text record），向因特网上的其他资源提供有关该域的信息。它是一种灵活的 DNS 记录类型，可根据具体内容提供多种用途。TXT 记录的一个常见用途是在域名服务器上创建 SPF 记录

## SPF

SPF 记录或称发送方政策框架记录（Sender Policy Framework record），列出了域或子域所指定的邮件服务器。它有助于确定邮件服务器的合法性，并减少欺骗的可能性（当有人伪造电子邮件的标题，使其看起来像是来自您的域时）。垃圾邮件发送者有时会尝试这样做以绕过过滤器。

## DKIM

DKIM 记录或称域名密匙确认邮件记录（`Domainkeys Identified Mail record`），它显示用于验证已经签署了 DKIM 协议的消息的公钥。这种做法提高了检查邮件可靠性的能力。一个典型的 DKIM 记录如下所示：

```s
selector1._domainkey.example.com        TXT     k=rsa;p=J8eTBu224i086iK
```

DKIM 记录以文本记录作为实现。该记录必须是为子域创建的记录，它具有唯一对应于键的一个选择器，然后便是句点（.），紧跟着是_domainkey.example.com。其类型为 TXT，值则包含键的类型，后面跟着实际键值。

## SRV

SRV记录： 添加服务记录服务器服务记录时会添加此项，SRV记录了哪台计算机提供了哪个服务。格式为：服务的名字.协议的类型（例如：_example-server._tcp）。

SRV 记录或称服务记录（Service record）将运行在您的域或子域上的指定服务匹配到一个目标与。这允许您将特定服务（如即时消息）的流量定向到另一台服务器。典型的 SRV 记录如下所示：

```s
_service._protocol.example.com  SRV     10      0       5060    service.example.com
```

下面分别解释 SRV 记录中的元素：

* **服务**：服务名称必须以下划线（_）开头，随后紧跟句点（.）。该服务可能类似于_xmpp。
* **协议**：协议的名称必须以下划线（_）开头，随后紧跟句点（.）。该协议可能类似于_tcp。
* **域**：将接收此服务的原始流量的域名称。
* **优先级**：第一个数字（上例中为 10）允许您设置目标服务器的优先级。您可以使用不同的优先级设置不同的目标，这令您可以拥有该服务的备用服务器（或多个服务器）。较低的数字具有较高的优先级。
* **权重**：如果两个记录具有相同的优先级，则需要对比权重。
* **端口**：运行服务的 TCP 或 UDP 端口。
* **目标**：目标域或目标子域。此域必须具有解析为 IP 地址的 A 或 AAAA 记录。

## OPT

选项资源记录。可将一个 OPT 资源记录添加到 DNS 请求或响应的附加数据部分。OPT 资源记录属于特定传输层消息（例如，UDP），不属于实际 DNS 数据。每条消息只允许具有一个 OPT 资源记录，但不是必需选项。

这是一个 `伪DNS记录类型`以支持 EDNS。EDNS：扩展DNS机制EDNS(Extension Mechanisms for DNS)。
