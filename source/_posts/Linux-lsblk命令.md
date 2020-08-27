---
title: Linux lsblk命令
date: 2020-08-27 09:38:04
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 简介

`lsblk` 命令用于列出所有可用块设备的信息，而且还能显示他们之间的依赖关系，但是它不会列出 `RAM盘` 的信息。块设备有 `硬盘`，`闪存盘`，`cd-ROM` 等等。`lsblk` 命令包含在 `util-linux-ng` 包中，现在该包改名为 `util-linux`。这个包带了几个其它工具，如`dmesg`。要安装`lsblk`，请在此处下载 `util-linux`包。Fedora用户可以通过命令 `sudo yum install util-linux-ng` 来安装该包。

选项：

```sh
-a, --all            显示所有设备。
-b, --bytes          以bytes方式显示设备大小。
-d, --nodeps         不显示 slaves 或 holders。
-D, --discard        print discard capabilities。
-e, --exclude <list> 排除设备 (default: RAM disks)。
-f, --fs             显示文件系统信息。
-h, --help           显示帮助信息。
-i, --ascii          use ascii characters only。
-m, --perms          显示权限信息。
-l, --list           使用列表格式显示。
-n, --noheadings     不显示标题。
-o, --output <list>  输出列。
-P, --pairs          使用key="value"格式显示。
-r, --raw            使用原始格式显示。
-t, --topology       显示拓扑结构信息。
```

显示说明：

* **NAME**：这是块设备名。
* **MAJ**: MIN：本栏显示主要和次要设备号。
* **RM**：本栏显示设备是否可移动设备。注意，在本例中设备sdb和sr0的RM值等于1，这说明他们是可移动设备。
* **SIZE**：本栏列出设备的容量大小信息。例如298.1G表明该设备大小为298.1GB，而1K表明该设备大小为1KB。
* **RO**：该项表明设备是否为只读。在本案例中，所有设备的RO值为0，表明他们不是只读的。
* **TYPE**：本栏显示块设备是否是磁盘或磁盘上的一个分区。在本例中，sda和sdb是磁盘，而sr0是只读存储（rom）。
* **MOUNTPOINT**：本栏指出设备挂载的挂载点。

## 实例

```sh
# 默认选项不会列出所有空设备，使用 -a 显示空设备。
lsblk -a

# 列出一个特定设备的拥有关系，同时也可以列出组和模式.
lsblk -m

# 该命令也可以只获取指定设备的信息，
# 可以通过在提供给lsblk命令的选项后指定设备名来实现
# 以字节显示你的磁盘驱动器大小
lsblk -b /dev/sda
# 等价于
lsblk --bytes /dev/sda

# 获取SCSI设备的列表
lsblk -S

# lsblk列出SCSI设备，而-s是逆序选项（将设备和分区的组织关系逆转过来显示）
lsblk -s
```

参考：

[Linux lsblk命令](https://man.linuxde.net/lsblk)