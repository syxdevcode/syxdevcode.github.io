---
title: Python3-网络编程
date: 2021-04-29 22:03:23
tags:
- Python3
- Python
categories: Python3
---

## 简介

网络编程有一个重要的概念 socket（套接字），应用程序可以通过它发送或接收数据，套接字允许应用程序将 I/O 插入到网络中，并与网络中的其他应用程序进行通信。

Python 提供了如下两个 socket 模块：

* **Socket** 提供了标准的 BSD Sockets API，可以访问底层操作系统 Socket 接口的全部方法。
* **SocketServer** 提供了服务器中心类，可以简化网络服务器的开发。

## API 介绍

Python 中通过 socket() 函数来创建套接字对象，具体格式如下：

```py
socket.socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)
```

* family：套接字协议族，可以使用 AF_UNIX（只能用于单一的 Unix 系统进程间通信）、AF_INET（服务器之间网络通信）
* type：套接字类型，可以使用 SOCK_STREAM（面向连接的）、SOCK_DGRAM（非连接的）

Socket服务端方法：

|方法 |	描述 |
|
bind(address)	将套接字绑定到地址，在 AF_INET 下以元组 (host,port) 的形式表示地址
listen([backlog])	开始监听 TCP 传入连接，backlog 指定在拒绝连接之前，操作系统可以挂起的最大连接数量，至少为1，大部分应用程序设为 5 就可以了
accept()	接受 TCP 连接并返回 (conn,address)，conn 是新的套接字对象，可以用来接收、发送数据，address 是连接客户端的地址



参考：

[Python 进阶（十）：网络编程](https://ityard.blog.csdn.net/article/details/104661342)