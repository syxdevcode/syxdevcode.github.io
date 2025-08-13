---
title: WPAD(Web代理自动发现协议)
date: 2022-05-10 14:29:55
tags:tags:
- PAC
- WPAD
categories:
- 计算机基础
---

## WPAD简介

WPAD（`Web Proxy Auto-Discovery Protocol`） 是 Web 代理自动发现协议的简称，该协议的功能是可以使局域网中用户的浏览器可以自动发现内网中的代理服务器，并使用已发现的代理服务器连接互联网或者企业内网。WPAD 支持所有主流的浏览器，从 IE 5.0 开始就已经支持了代理服务器自动发现/切换的功能，苹果公司考虑到 WPAD 的安全风险，在包括 OSX 10.10 及之后版本的操作系统中的 Safari 浏览器将不再支持 PAC 文件的解析。

## WPAD工作原理

当系统开启了代理自动发现功能后，用户使用浏览器上网时，浏览器就会在当前局域网中自动查找代理服务器，如果找到了代理服务器，则会从代理服务器中下载一个名为 PAC（`Proxy Auto-Config`） 的配置文件。该文件中定义了用户在访问一个 URL 时所应该使用的代理服务器。浏览器会下载并解析该文件，并将相应的代理服务器设置到用户的浏览器中。

<!--more-->
## PAC文件

{% post_link 'PAC(代理自动配置)' %}

## Windows中的WPAD

在 Windows 系统中，从 IE5.0 开始就支持了 WPAD，并且 Windows 系统默认是开启了 WPAD 功能的。
可以在 IE浏览器的 `Internet 选项 — 连接 选项卡 — 局域网设置 — 自动检测设置` 中看到，系统默认是勾选此功能的。

![Snipaste_2022-05-10_14-53-23.png](/img/Snipaste_2022-05-10_14-53-23.png)

另外， Windows 系统从 IE 5.5 开始支持“自动代理结果缓存”功能，并默认开启了此功能，此功能的机制为每当客户端的浏览器连接成功 HTTP 代理服务器，都会更新 ARP 缓存，所以在客户端浏览器再次连接代理服务器也就是在再次调用 `FindProxyForURL()` 函数时，会先检查 ARP 缓存列表中，是否存在要连接的 HTTP 代理服务器地址。所以此功能的目的就是缩减系统获取分配对象的开销。

可以使用如下操作关闭此功能：

**方法1：修改注册表**
可以使用下面的注册表项禁用“自动代理结果缓存”：
`HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\CurrentVersion\Internet Settings`
将 `EnableAutoproxyResultCache`（如果不存在请手动创建，类型为 REG_DWORD） 设置为 0 或者 1 。
0 = 禁用缓存；1 = 启用自动代理缓存（这是默认设置）

**方法 2：修改组策略设置**
在组策略对象编辑器中的 `用户配置—管理模板—Windows组件—Internet Explorer`中，启用 `禁用缓存自动代理脚本` 即可。

## WPAD实现方式

WPAD 实现的方式有两种，DHCP 和 DNS。

### 使用DHCP服务器配置WPAD

DHCP 是 `Dynamic Host Configuration Protocol` 的缩写即 `动态主机配置协议`，它是一个用于局域网的网络协议，处于 OSI 的应用层，所使用的传输协议为 UDP。DHCP 的主要功能是动态分配，当然不仅仅是 IP 地址，也包括一些其他信息，如子网掩码等，也包括本文所讲的 WPAD，这些额外的信息都是通过 DHCP 协议中的 `DHCP options` 字段传输的。

当使用 DHCP 服务器配置 WPAD 时， DHCP 协议将会有所改变，具体的改变可以在 RFC 2131 中看到，增加了 DHCPINFORM 消息，此消息用于客户端请求本地配置参数，所以客户端在请求 WPAD 主机时就会发送 DHCPINFORM 请求消息，之后 DHCP 服务器会应答 DHCPACK 确认消息，此消息中的 DHCP Options 字段里就包含 DHCP 的 252 选项即 WPAD 代理服务器的 PAC 文件地址。

在目前的大多数内网中已经不再使用 DHCP 服务器来进行对客户端的 WPAD 的配置了，而是采用了较为简单的方式如 DNS 服务器。

### 利用DNS配置WPAD

利用 DNS 配置 WPAD 的方式本质上还是利用了 Windows 系统的名称解析机制。

**利用 DNS 配置 WPAD 的原理如下：**

客户端主机向 DNS 服务器发起了 `WPAD＋X` 的查询请求。如果客户端主机是处于域环境下时，发起的 WPAD+X 的查询请求为 “WPAD.当前域的域名”。DNS 服务器对 WPAD 主机的名称进行解析返回 WPAD 主机的 IP 地址，客户端主机通过 WPAD 主机的 IP 的 80 端口下载并解析 PAC 文件。
利用 DNS 进行 WPAD 的配置，网络管理员只需要在 DNS 服务器中添加 WPAD 主机的解析记录即可。
PS：在工作组环境中，客户端主机执行 WPAD 功能时，就会遵循 Windows 系统的名称解析顺序，查询的名称均为 “WPAD”，如果 OS 版本为 Vista 之后的（包括 Vista）顺序为：DNS => LLMNR => NBNS，反之则为 DNS => NBNS。

参考：

[WPAD 协议分析及内网渗透利用](https://xz.aliyun.com/t/1739/#toc-7)