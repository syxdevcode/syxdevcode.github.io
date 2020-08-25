---
title: linux Xinetd服务
date: 2020-08-21 17:50:29
tags:
- Linux
- CentOS7
- Linux服务
categories:
- Linux服务
---

## Linux进程概念


## 简介

`xinetd` （ `extended internet daemon` ）是新一代的网络守护进程服务程序，又叫超级 `Internet` 服务器,常用来管理多种轻量级 `Internet` 服务。
xinetd提供类似于 `inetd+tcp_wrapper` 的功能，但是更加强大和安全。

基于 `xinetd` 的服务有启动管理和自启动管理之分，而且不管是启动管理还是自启动管理，都只有一种方法，相比独立的服务简单一些。

基于 `xinetd` 的服务没有自己独立的启动脚本程序，是需要依赖 `xinetd` 的启动脚本来启动的。`xinetd` 本身是独立的服务，所以 `xinetd` 服务自己的启动方法和独立服务的启动方法是一致的。

## 特点

### 强大的存取控制功能

* 内置对恶意用户和善意用户的差别待遇设定。
* 使用libwrap支持，其效能更甚于tcpd。
* 可以限制连接的等级，基于主机的连接数和基于服务的连接数。
*  设置特定的连接时间。
* 将某个服务设置到特定的主机以提供服务。

### 有效防止DoS攻击

* 可以限制连接的等级。
* 可以限制一个主机的最大连接数，从而防止某个主机独占某个服务。
* 可以限制日志文件的大小，防止磁盘空间被填满。

### 强大的日志功能

* 可以为每一个服务就syslog设定日志等级。
* 如果不使用syslog，也可以为每个服务建立日志文件。
* 可以记录请求的起止时间以决定对方的访问时间。
* 可以记录试图非法访问的请求。

### 转向功能

可以将客户端的请求转发到另一台主机去处理。

### 支持IPv6

xinetd自 `xinetd 2.1.8.8pre*` 起的版本就支持IPv6，可以通过在./configure脚本中使用 `with-inet6 capability` 选项来完成。注意，要使这个生效，核心和网络必须支持IPv6。当然IPv4仍然被支持。

### 与客户端的交互功能

无论客户端请求是否成功，xinetd都会有提示告知连接状态。

##  Xinetd的缺点

当前，它最大的缺点是对RPC支持的不稳定性，但是可以启动protmap，使它与xinetd共存来解决这个问题。

## 使用xinetd启动守护进程

原则上任何系统服务都可以使用 `xinetd` ，然而最适合的应该是那些常用的网络服务，同时，这个服务的请求数目和频繁程度不会太高。像 `DNS` 和 `Apache` 就不适合采用这种方式，而像 `FTP`、`Telnet`、`SSH` 等就适合使用 `xinetd` 模式，系统默认使用 `xinetd` 的服务可以分为如下几类。

* 标准Internet服务：telnet、ftp。
* 信息服务：finger、netstat、systat。
* 邮件服务：imap、imaps、pop2、pop3、pops。
* RPC服务：rquotad、rstatd、rusersd、sprayd、walld。
* BSD服务：comsat、exec、login、ntalk、shell、talk。
* 内部服务：chargen、daytime、echo、servers、services、time。
* 安全服务：irc。
* 其他服务：name、tftp、uucp。

具体可以使用 `xinetd` 的服务在 `/etc/services` 文件中指出。这个文件的节选内容如下所示：

```sh
# /etc/services:
# $Id: services,v 1.40 2004/09/23 05:45:18 notting Exp $
# service-name  port/protocol  [aliases ...]   [# comment]
tcpmux      1/tcp               # TCP port service multiplexer
tcpmux      1/udp               # TCP port service multiplexer
rje     5/tcp                   # Remote Job Entry
rje     5/udp                   # Remote Job Entry
echo        7/tcp
echo        7/udp
discard     9/tcp       sink null
discard     9/udp       sink null
```

Internet网络服务文件中，记录网络服务名和它们对应使用的端口号及协议。文件中的每一行对应一种服务，它由4个字段组成，中间用Tab键或空格键分隔，分别表示 `服务名称`、 `使用端口`、 `协议名称` 及 `别名` 。

在一般情况下，不要修改该文件的内容，因为这些设置都是Internet标准的设置。一旦修改，可能会造成系统冲突，使用户无法正常访问资源。Linux系统的端口号的范围为 `0~65535`，不同范围的端口号有不同的意义。

* 0：不使用。
* 1~1 023：系统保留，只能由root用户使用。
* 1 024~4 999：由客户端程序自由分配。
* 5 000~65 535：由服务器程序自由分配。

## /etc/xinetd.conf和/etc/xinetd.d/*

###  /etc/xinetd.conf

xinetd的配置文件是/etc/xinetd.conf，但是它只包括几个默认值及/etc/xinetd.d目录中的配置文件。如果要启用或禁用某项xinetd服务，编辑位于/etc/xinetd.d目录中的配置文件。

`/etc/xinetd.conf` 有许多选项，下面是RHEL 4.0的 `/etc/xinetd.conf` 。

```sh
# Simple configuration file for xinetd
# Some defaults, and include /etc/xinetd.d/
defaults
{
    instances             = 60
    log_type               = SYSLOG authpriv
    log_on_success       = HOST PID
    log_on_failure       = HOST
    cps                   = 25 30
}
includedir /etc/xinetd.d
```

* instances = 60：表示最大连接进程数为60个。
* log_type = SYSLOG authpriv：表示使用syslog进行服务登记。
* log_on_success= HOST PID：表示设置成功后记录客户机的IP地址的进程ID。
* log_on_failure = HOST：表示设置失败后记录客户机的IP地址。
* cps = 25 30：表示每秒25个入站连接，如果超过限制，则等待30秒。主要用于对付拒绝服务攻击。
* includedir /etc/xinetd.d：表示告诉xinetd要包含的文件或目录是/etc/xinetd.d。

### /etc/xinetd.d/*

下面以 `/etc/xinetd.d/` 中的一个文件（rsync）为例。

```sh
service rsync
{
    disable = yes
    socket_type      = stream
    wait              = no
    user              = root
    server           = /usr/bin/rsync
    log_on_failure += USERID
}
```

下面说明每一行选项的含义。

* disable = yes：表示禁用这个服务。
* socket_type = stream：表示服务的数据包类型为stream。
* wait = no：表示不需等待，即服务将以多线程的方式运行。
* user = root：表示执行此服务进程的用户是root。
* server = /usr/bin/rsync：启动脚本的位置。
* log_on_failure += USERID：表示设置失败时，UID添加到系统登记表。

### 配置xinetd

格式：

`/etc/xinetd.conf` 中的每一项具有下列形式：

```sh
service service-name
{
……
}
```

其中 `service` 是必需的关键字，且属性表必须用大括号括起来。每一项都定义了由 `service-name` 定义的服务。

`service-name` 是任意的，但通常是标准网络服务名，也可增加其他非标准的服务，只要它们能通过网络请求激活，包括 `localhost` 自身发出的网络请求。有很多可以使用的属性，稍后将描述必需的属性和属性的使用规则。

操作符可以是=、+=或-=。所有属性可以使用=，其作用是分配一个或多个值，某些属性可以使用+=或-=，其作用分别是将其值增加到某个现存的值表中，或将其值从现存值表中删除。

### 配置文件

相关的配置文件如下：

```sh
/etc/xinetd.conf
/etc/xinetd.d/*   //该目录下的所有文件
/etc/hosts.allow
/etc/hosts.deny
```

### disabled与enabled

前者的参数是禁用的服务列表，后者的参数是启用的服务列表。他们的共同点是格式相同（属性名、服务名列表与服务中间用空格分开，例如 `disabled = in.tftpd in.rexecd` ），此外，它们都是作用于全局的。如果在disabled列表中被指定，那么无论包含在列表中的服务是否有配置文件和如何设置，都将被禁用；如果enabled列表被指定，那么只有列表中的服务才可启动，如果enabled没有被指定，那么disabled指定的服务之外的所有服务都可以启动。

### 注意问题

* 在重新配置的时候，下列的属性不能被改变：`socket_type`、`wait`、`protocol`、`type`；
* 如果 `only_from` 和 `no_access` 属性没有被指定（无论在服务项中直接指定还是通过默认项指定），那么对该服务的访问IP将没有限制；
* 地址校验是针对IP地址而不是针对域名地址。

### xinetd防止拒绝服务攻击（Denial of Services）的原因

* **限制同时运行的进程数**

通过设置instances选项设定同时运行的并发进程数：

`instances＝20`

当服务器被请求连接的进程数达到20个时，xinetd将停止接受多出部分的连接请求。直到请求连接数低于设定值为止。

* **限制一个IP地址的最大连接数**

通过限制一个主机的最大连接数，从而防止某个主机独占某个服务。

`per_source＝5`

这里每个IP地址的连接数是5个。

* **限制日志文件大小，防止磁盘空间被填满**

许多攻击者知道大多数服务需要写入日志。入侵者可以构造大量的错误信息并发送出来，服务器记录这些错误，可能就造成日志文件非常庞大，甚至会塞满硬盘。同时会让管理员面对大量的日志，而不能发现入侵者真正的入侵途径。因此，限制日志文件大小是防范拒绝服务攻击的一个方法。

`log_type FILE.1 /var/log/myservice.log 8388608 15728640`

这里设置的日志文件FILE.1临界值为8MB，到达此值时，syslog文件会出现告警，到达15MB，系统会停止所有使用这个日志系统的服务。

* **限制负载**

xinetd还可以使用限制负载的方法防范拒绝服务攻击。用一个浮点数作为负载系数，当负载达到这个数目的时候，该服务将暂停处理后续的连接。

`max_load = 2.8`

上面的设定表示当一项系统负载达到2.8时，所有服务将暂时中止，直到系统负载下降到设定值以下。

说明  要使用这个选项，编译时应加入 `--with-loadavg` ，xinetd将处理 `max-load` 配置选项，从而在系统负载过重时关闭某些服务进程，来实现防范某些拒绝服务攻击。

* **限制所有服务器数目（连接速率）**

xinetd可以使用cps选项设定连接速率，下面的例子：

`cps = 25 60`

上面的设定表示服务器最多启动25个连接，如果达到这个数目将停止启动新服务60秒。在此期间不接受任何请求。

* **限制对硬件资源的利用**

通过rlimit_as和rlimit_cpu两个选项可以有效地限制一种服务对内存、中央处理器的资源占用：

```sh
rlimit_as = 8M
rlimit_cpu=20
```

上面的设定表示对服务器硬件资源占用的限制，最多可用内存为8MB，CPU每秒处理20个进程。

xinetd的一个重要功能是它能够控制从属服务可以利用的资源量，通过它的以上设置可以达到这个目的，有助于防止某个xinetd服务占用大量资源，从而导致“拒绝服务”情况的出现。

参考：

[什么是Xinetd](http://blog.chinaunix.net/uid-21411227-id-1826885.html)

[CentOS 7.3 Xinetd服务的安装与配置](https://blog.51cto.com/13525470/2060765)

[Linux之Xinetd服务介绍](https://blog.csdn.net/lzghxjt/article/details/83018710)

[Linux基于xinetd服务的管理方法详解](http://c.biancheng.net/view/1054.html)