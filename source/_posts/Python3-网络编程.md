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

## Python Internet 模块

以下列出了 Python 网络编程的一些重要模块：

|协议 |	功能用处 |	端口号 | Python 模块 |
| :---|  :---   |   :---    | :---   |
|HTTP|	网页访问|	80	| httplib, urllib, xmlrpclib |
|NNTP|	阅读和张贴新闻文章，俗称为 帖子| 	119	|nntplib |
|FTP|	文件传输|	20	| ftplib, urllib |
|SMTP|	发送邮件|	25	| smtplib |
|POP3|	接收邮件|	110	| poplib |
|IMAP4|	获取邮件|	143	| imaplib |
|Telnet|	命令行|	23	| telnetlib |
|Gopher|	信息查找	|70	| gopherlib, urllib |

<!--more-->

## API 介绍

Python 中通过 socket() 函数来创建套接字对象，具体格式如下：

```py
socket.socket(family=AF_INET, type=SOCK_STREAM, proto=0, fileno=None)
```

* family：套接字协议族，可以使用 `AF_UNIX`（只能用于单一的 Unix 系统进程间通信）、`AF_INET`（服务器之间网络通信）
* type：套接字类型，可以使用 `SOCK_STREAM`（面向连接的）、`SOCK_DGRAM`（非连接的）
* protocol: 一般不填默认为0.

Socket 服务端方法：

|方法 |	描述 |
|:---: | :---|
|bind(address) |	将套接字绑定到地址，在 AF_INET 下以元组 (host,port) 的形式表示地址 |
|listen([backlog]) |	开始监听 TCP 传入连接，backlog 指定在拒绝连接之前，操作系统可以挂起的最大连接数量，至少为1，大部分应用程序设为 5 就可以了 |
|accept()	| 接受 TCP 连接并返回 (conn,address)，conn 是新的套接字对象，可以用来接收、发送数据，address 是连接客户端的地址 |

Socket 客户端方法：

|方法 |	描述 |
|:---: | :---|
|connect(address) |	连接到 address 处的套接字，格式一般为元组 (hostname,port)，如果连接出错，返回 socket.error 错误 |
|connect_ex(address) |	功能与 connect(address) 相同，但是成功返回 0，失败返回 errno 的值 |


套接字对象公用方法：

|方法 |	描述 |
|:---: | :---|
|recv(bufsize[, flags])|接受 TCP 套接字的数据，数据以字符串形式返回，bufsize 指定要接收的最大数据量，flag 提供有关消息的其他信息，通常可以忽略|
|send(bytes[, flags])|发送 TCP 数据，将 string 中的数据发送到连接的套接字，返回值是要发送的字节数量，该数量可能小于 string 的字节大小|
|sendall(bytes[, flags])|完整发送 TCP 数据，将 string 中的数据发送到连接的套接字，但在返回之前会尝试发送所有数据，成功返回 None，失败则抛出异常|
|recvfrom(bufsize[, flags])|	接受 UDP 套接字的数据，与 recv() 类似，但返回值是 (data,address)，其中 data 是包含接收数据的字符串，address 是发送数据的套接字地址|
|sendto(bytes, flags, address)	|发送 UDP 数据，将数据发送到套接字，address 是形式为 (ipaddr,port) 的元组，指定远程地址，返回值是发送的字节数|
|close()|	关闭套接字|
|getpeername()|	返回连接套接字的远程地址，类型通常是元组 (ipaddr,port)|
|getsockname()|	返回套接字自己的地址，通常是一个元组 (ipaddr,port)|
|setsockopt(level,optname,value)|	设置给定套接字选项的值|
|getsockopt(level, optname[, buflen])|	返回套接字选项的值|
|settimeout(value)|	设置套接字操作的超时时间，单位是秒|
|gettimeout()|	返回当前超时时间|
|fileno()|	返回套接字的文件描述符|
|setblocking(flag)|	如果 flag 为 0，则将套接字设为非阻塞模式，否则将套接字设为阻塞模式（默认值）；非阻塞模式下，如果调用 recv() 没有发现任何数据或 send() 调用无法立即发送数据，那么将引起 socket.error 异常|
|makefile()|	创建一个与该套接字相关连的文件|

## TCP 方式

先运行服务端代码，再运行客户端代码

服务端基本思路：

* 创建套接字，绑定套接字到 IP 与端口
* 监听连接
* 不断接受客户端的连接请求
* 接收请求的数据，并向对方发送响应数据
* 传输完毕后，关闭套接字

代码实现如下：

```py
import socket

# 创建套接字
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 绑定地址
s.bind(('127.0.0.1', 6666))
# 监听连接
s.listen(5)
while True:
    print('等待客户端发送信息...')
    # 接收连接
    sock, addr = s.accept()
    # 接收请求数据
    data = sock.recv(1024).decode('utf-8')
    print('服务端接收请求数据：' + data)
    # 发送响应数据
    sock.sendall(data.upper().encode('utf-8'))
    # 关闭
    sock.close()
```

客户端基本思路：

* 创建套接字，连接服务端
* 连接后发送、接收数据
* 传输完毕后，关闭套接字

具体代码实现如下：

```py
import socket

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 连接服务端
s.connect(('127.0.0.1', 6666))
# 向服务端发送数据
s.sendall(b'hello')
# 接受服务端响应数据
data = s.recv(1024)
print('客户端接收响应数据：' + data.decode('utf-8'))
# 关闭
s.close()
```

## UDP方式

服务端基本思路：

* 创建套接字，绑定套接字到 IP 与端口
* 接收客户端请求的数据
* 向客户端发送响应数据

代码实现如下：

```py
import socket

# 创建套接字
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
# 绑定地址
s.bind(('127.0.0.1', 6666))
while True:
    # 接收数据
    data, addr = s.recvfrom(1024)
    print('服务端接收请求数据：' + data.decode('utf-8'))
    # 响应数据
    s.sendto(data.decode('utf-8').upper().encode('utf-8'), addr)
```

客户端基本思路：

* 创建套接字
* 向服务端发送数据
* 接受服务端响应数据

```py
import socket

# 创建套接字
s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
# 向服务端发送数据
s.sendto(b'hello', ('127.0.0.1', 6666))
# 接受服务端响应数据
data = s.recv(1024).decode('utf-8')
print('客户端接收响应数据：' + data)
# 关闭
s.close()
```

参考：

[Python 进阶（十）：网络编程](https://ityard.blog.csdn.net/article/details/104661342)