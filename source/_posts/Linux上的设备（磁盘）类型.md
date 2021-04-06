---
title: Linux上的设备（磁盘）类型
date: 2020-08-28 13:29:16
tags:
- 计算机基础
- 硬盘
- Linux
- LVM
- 计算机科学
categories: Linux 磁盘
---

## Linux 上的设备 （device）

Linux 操作系统中，各种设备驱动（`device driver`）通过设备控制器（`device controller`）来管理各种设备（`device`），其关系如下图所示：

![697113-20160802132422028-1983172341.jpg](/img/697113-20160802132422028-1983172341.jpg)
<!--more-->
这些设备之中，

* 受同一个 `device driver` 管理的设备都有相同的 `major number`，这个数字可以看作设备的类别号码，被内核用于识别一类设备
* 受同一个 `device driver` 管理的同一类设备中的每一个设备都有不同的 `minor number`，这个数字可以看作设备编号，被设备驱动用来识别每个设备

major：主要的；重要的；大的；严重；
minor：较小的；次要的；副修；

设备驱动主要有三大类：

* 面向包的网络设备驱动（package oriented network device driver）
* 面向块的存储设备驱动（block oriented storage device driver），提供缓冲式（buffered）的设备访问。
* 面向字节的字符设备驱动 （byte oriented char device driver），有时也称为裸设备（raw devices），提供非缓冲的直接的设备访问（unbuffered direct access），比如串口设备，摄像头，声音设备等。

实际上，除了网络设备和存储设备以外的其它设备都是某种字符设备。
除此以外，还有一类设备，称为伪设备（pseudo device），它们是软件设备。Linux 上的 device 不一定要有硬件设备，比如 `/dev/null`, `/dev/zero` 等。

`字符设备` 和 `块设备` 的区别：

* 块设备只能以块为单位接受输入和返回输出，而字符设备则以字节为单位。大多数设备是字符设备，因为它们不需要缓冲而且不以固定块大小进行操作。
* 块设备对于 `I/O` 请求有对应的缓冲区，因此它们可以选择以什么顺序进行响应，字符设备无须缓冲且被直接读写。对于存储设备而言调 读写的顺序作用巨大，因为在读写连续的扇区比分离的扇区更快。
* 字符设备只能被顺序读写，而块设备可以随机访问。虽然块设备可随机访问，但是对于磁盘这类机械设备而言，顺序地组织块设备的访问可以提高性能。

用户空间的各种应用是通过 `device driver` 来操作设备的：

![697113-20160802132825043-1473964830.jpg](/img/697113-20160802132825043-1473964830.jpg)

如果再详细一些就是这样的：

![697113-20160802144539497-1862957917.jpg](/img/697113-20160802144539497-1862957917.jpg)

图上可以看出：

* 网络设备驱动之上，分别有包调度器（`packet scheduler`），网络协议层（`network protocols`），`NetFilter` （防火墙）和 `scoket` 层，其中，网络设备驱动以 `socket` 作为应用层的接口
* 块设备驱动之上，分别有 `I/O Scheduler`，通用块层（`generic block layer`）和文件系统，其中，块设备驱动以设备文件 （`device file`）作为应用层的接访问口
字符设备驱动之上，分别有 `Line discipline` 和 `terminals`，其中，`terminals` 作为和应用的访问接口。

Linux 系统中 `一切皆文件`。每个设备，在 /dev 目录中都有对一个设备文件（`device file`），比如 `/dev/sda` 表示第一个 SCSI/IDE 盘，`/dev/vda` 表示第一个 virtio 磁盘。应用程序通过访问这些设备文件像操作文件一样来访问这些设备，可以使用的接口包括：

* int open(const char *path, int oflag, ... )
* int close(int fd);
* ssize_t write(int fd, const void *buf, size_t nbyte)
* ssize_t read(int fd, void *buf, size_t nbyte)
* int ioctl(int d, int request, ...)

在 Linux 系统上，设备驱动可以被动态加载和删除

```sh
lsmod - 列出当前已经被加载的模块
insmod <module_file> - insert/load 指定的模块文件
modprobe <module> - insert/load 指定的 module，以及所有依赖
rmmod <module> - remove/unload 指定的module
```

## Linux 设备的 major 和 minor number

### 用 ls 获取

```sh
cd /dev
ls -l
[root@host-192-125-30-71 dev]# ls -l
总用量 0
crw-------  1 root root     10, 235 9月  18 2020 autofs
drwxr-xr-x  2 root root         200 12月 31 16:08 block
```

* 以 `c` 开头的一行表示该设备是一个字符设备，以 `b` 开头的行表示这是一个块设备。
* 10,235 这两个数字中，前面的 10 表示 `major number`，后面的 235 表示 `minor number`。

### major 和 minor 值的设置

历史上，设备的 `major number` 采用的是注册制，各设备厂家在 `http://www.lanana.org/` 中注册他们的设备所使用的 `major number`。从 `http://www.lanana.org/docs/device-list/devices-2.6+.txt` 中还可以看出来 linux 2.6 内核中所分配的静态 `major numbers`。

但是，现在，这个注册网站已经没有人维护了，取而代之的是动态分配制度。分配者是linux 内核的 udev 模块。它将保证在本系统中，`<major number>：<minor number>` 的组合是唯一的，而在这个范围之外，它不会保证其惟一性。一旦分配好了后，你就可以从 `/proc/devices` 文件中读出所分配的 `major numbers`，比如：

```sh
[root@root dev]# cat /proc/devices
Character devices:
  1 mem
  4 /dev/vc/0
  4 tty
  4 ttyS
  5 /dev/tty
  5 /dev/console
  5 /dev/ptmx
```

## 根据 major 和 minior numbers 识别磁盘类型

根据以下步骤来识别磁盘类型：

**（1）使用 stat 命令获取设备文件的 major 和 minor numbers。注意结果是16进制。**

```sh
cd /dev
stat -c %T /dev/vda #minor number
0

stat -c %T /dev/vdb
10

stat -c %T /dev/sda
0

stat -c %t /dev/vda #major number
fd

stat -c %t /dev/vdb
fd

stat -c %t /dev/sda
```

**将16进制数字转化为10进制，并拼接字符串 /sys/dev/block/$decmajor:$minor/device/driver**

`/sys/dev/block/8:0/device/driver`

**调用 readlink -f 命令，获取 device driver**

```sh
readlink -f /sys/dev/block/8:0/device/driver
/sys/bus/scsi/drivers/sd
```

从输出可以看出 `/dev/sda` 是 sd 即 SCSI 类型的设备。 

常见的命名：

* fd：软驱
* hd：IDE 磁盘
* sd：SCSI 磁盘
* tty：terminals
* vd：virtio 磁盘

转载：

[识别 Linux上的设备（磁盘）类型](https://www.cnblogs.com/sammyliu/p/5729026.html)