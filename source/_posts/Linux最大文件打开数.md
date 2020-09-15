---
title: Linux最大文件打开数
date: 2020-08-04 13:57:52
tags:
- Linux
- CentOS7
categories:
- Linux
---

## 简介

linux最大打开文件句柄数，即打开文件数最大限制，就是规定的单个进程能够打开的最大文件句柄数量（Socket连接也算在里面，默认大小1024）

在Linux下有时会遇到Socket/File : Can't open so many files的问题。其实Linux是有文件句柄限制的，而且Linux默认一般都是1024（阿里云主机默认是65535）。在生产环境中很容易到达这个值，因此这里就会成为系统的瓶颈。

## 相关命令

### 查看命令

**用户级的最大限制**

`ulimit -a` 或者 `ulimit -n`

`ulimit -n`，默认是1024，向阿里云华为云这种云主机一般是65535。

```sh
[root@host-192-125-30-59 ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 256967
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 256967
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
[root@host-192-125-30-59 ~]# ulimit -n
1024
```

open files （-n） 1024 是linux操作系统对一个进程打开的文件句柄数量的限制（也包含打开的套接字数量）


| 选项 [options]   |  含义 |	例子 |
| --- |--- | --- |
 | -H	 | 设置硬资源限制，一旦设置不能增加。	 | ulimit – Hs 64；限制硬资源，线程栈大小为 64K。 | 
 | -S	 | 设置软资源限制，设置后可以增加，但是不能超过硬资源设置。	 | ulimit – Sn 32；限制软资源，32 个文件描述符。 | 
 | -a	 | 显示当前所有的 limit 信息。 | 	ulimit – a；显示当前所有的 limit 信息。
 | -c	 | 最大的 core 文件的大小， 以 blocks 为单位。 | 	ulimit – c unlimited； 对生成的 core 文件的大小不进行限制。
 | -d	 | 进程最大的数据段的大小，以 Kbytes 为单位。 | 	ulimit -d unlimited；对进程的数据段大小不进行限制。
 | -f	 | 进程可以创建文件的最大值，以 blocks 为单位。 | 	ulimit – f 2048；限制进程可以创建的最大文件大小为 2048 blocks。
 | -l	 | 最大可加锁内存大小，以 Kbytes 为单位。 | 	ulimit – l 32；限制最大可加锁内存大小为 32 Kbytes。
 | -m	 | 最大内存大小，以 Kbytes 为单位。 | 	ulimit – m unlimited；对最大内存不进行限制。
 | -n	 | 可以打开最大文件描述符的数量。 | 	ulimit – n 128；限制最大可以使用 128 个文件描述符。
 | -p	 | 管道缓冲区的大小，以 Kbytes 为单位。 | 	ulimit – p 512；限制管道缓冲区的大小为 512 Kbytes。
 | -s	 | 线程栈大小，以 Kbytes 为单位。	 | ulimit – s 512；限制线程栈的大小为 512 Kbytes。
 | -t	 | 最大的 CPU 占用时间，以秒为单位。 | 	ulimit – t unlimited；对最大的 CPU 占用时间不进行限制。
 | -u	 | 用户最大可用的进程数。	 | ulimit – u 64；限制用户最多可以使用 64 个进程。
 | -v	 | 进程最大可用的虚拟内存，以 Kbytes 为单位。 | 	ulimit – v 200000；限制最大可用的虚拟内存为 200000 Kbytes。


**系统级的最大限制**

```sh
[root@host-192-125-30-59 ~]# cat /proc/sys/fs/file-max
6523120
```

`file-max` 是设置系统所有进程一共可以打开的文件数量 。同时一些程序可以通过setrlimit调用，设置每个进程的限制。如果得到大量使用完文件句柄的错误信息，是应该增加这个值。


### 查看某个进程的最大打开文件数和当前打开文件数

先找到该进程的进程号，然后查看

```sh
# 找到pid
ps -aux

# /proc/[pid]/limits和fd

cat /proc/[pid]/limits 显示当前进程的资源限制

ls /proc/[pid]/fd 是一个目录，包含进程打开文件的情况
```

实例：

```sh
[root@host-192-125-30-59 ~]# cat /proc/3289/limits| grep files
Max open files            1024                 4096                 files
[root@host-192-125-30-59 ~]# ll /proc/3289/fd | wc -l
7
```

ps：如果要查看某个进程的线程的详细信息，/proc/[pid]/task

### 修改最大限制

#### 用户级修改

**临时生效方法：（重启后失效）**

这个设置是暂时的保留。当退出Shell会话后，该值恢复原值。

```sh
ulimit -SHn 102400
```

ulimit 命令分软限制和硬限制，加-H就是硬限制，加-S就是软限制。默认显示的是软限制，如果运行ulimit 命令修改时没有加上-H或-S，就是两个参数一起改变。

硬限制就是实际的限制，而软限制是警告限制，它只会给出警告。

**用户级修改永久有效方式**

修改配置文件 `vim /etc/security/limits.conf`，加入：

```sh
* soft nofile 102400
* hard nofile 102400
```

或者使用命令：

```sh
echo "* soft nofile 65535" >> /etc/security/limits.conf
echo "* hard nofile 65535" >> /etc/security/limits.conf
```

`*` 表示所用的用户，但有的系统不认, 需要具体的用户名, 比如:

```sh
root soft nofile 65535
root hard nofile 65535
```

有些环境修改后，root用户需要重启机器才生效，而普通用户重新登录后即生效。

需要小心注意的是，有些环境上面这样做，可能导致无法ssh登录，所以在修改时最好打开两个窗口，万一登录不了还可自救。
 
#### 系统级的修改

其实上面的修改都是对一个进程打开的文件句柄数量的限制，我们还需要设置系统的总限制才可以。

假如，我们设置进程打开的文件句柄数是1024 ，但是系统总限制才500，所以所有进程最多能打开文件句柄数量500。从这里我们可以看出只设置进程的打开文件句柄的数量是不行的。所以需要修改系统的总限制才可以。

```sh
# 系统级修改临时生效方式：
echo 6523120 > /proc/sys/fs/file-max

# 系统级修改永久生效方式：
vim /etc/sysctl.conf
# 加入
fs.file-max=6523120
```

系统级永久生效方式修改后，重启服务器，才能生效。

查看系统级文件句柄修改，是否生效

```sh
sysctl -p
```

参考：

[Linux最大文件打开数](https://www.cnblogs.com/pangguoping/p/5791432.html)

[linux最大打开文件句柄数](https://www.cnblogs.com/xulan0922/p/11937472.html)