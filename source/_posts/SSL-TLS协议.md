---
title: SSL/TLS协议
date: 2018-07-09 11:07:56
tags:
- https
- 加密
- SSL/TLS
categories:
- SSL/TLS
---
# SSL/TLS协议

查看系统支持的SSL/TLS版本：

需要安装openssl.

```bash
openssl ciphers -v | awk '{print $2}' | sort | uniq
```

## HTTPS

HTTPS（Hyper Text Transfer Protocol Secure），即超文本传输安全协议，也称为http over tls等，是一种网络安全传输协议，其也相当于工作在七层的http，只不过是在会话层和表示层利用ssl/tls来加密了数据包，访问时以`https://`开头，默认443端口，同时需要证书，学习https的原理其实就是在学习ssl/tls的原理。

## SSL

SSL：（Secure Socket Layer，安全套接字层），位于可靠的面向连接的网络层协议和应用层协议之间的一种协议层。目的是为互联网通信提供安全及数据完整性保障，使用X.509认证，网景公司（Netscape）在1994年推出HTTPS协议，以SSL进行加密，这是SSL的起源。IETF将SSL进行标准化，1999年公布第一版TLS标准文件。SSL通过互相认证、使用数字签名确保完整性、使用加密确保私密性，以实现客户端和服务器之间的安全通讯。该协议由两层组成：SSL记录协议和SSL握手协议。SSL目前有三个版本，SSL1.0、SSL2.0、SSL3.0，因其存在严重的安全问题，大多数公司目前均已不在使用了。

参考：

[传输层安全性协议](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)

## TLS

TLS：(Transport Layer Security，传输层安全协议)，用于两个应用程序之间提供保密性和数据完整性。该协议由两层组成：TLS记录协议和TLS握手协议。TLS建立在SSL 3.0协议规范之上，是SSL 3.0的后续版本。两者差别极小，可以理解为SSL 3.1，它是写入了RFC的。TLS 1.0包括可以降级到SSL 3.0的实现，这削弱了连接的安全性。

版本：TLS 1.0，TLS 1.1，TLS 1.2，TLS 1.3（2018.3.21）。

参考：

[SSL与TLS的区别以及介绍](https://kb.cnblogs.com/page/197396/)