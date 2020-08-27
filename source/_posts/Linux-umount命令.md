---
title: Linux umount命令
date: 2020-08-27 13:50:07
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 简介

Linux umount命令用于卸除文件系统。
umount可卸除目前挂在Linux目录中的文件系统。

## 语法

```sh
umount [-ahnrvV] [-t <文件系统类型>] [文件系统]
```

参数：

* -a 卸除/etc/mtab中记录的所有文件系统。
* -h 显示帮助。
* -n 卸除时不要将信息存入/etc/mtab文件中。
* -r 若无法成功卸除，则尝试以只读的方式重新挂入文件系统。
* -t<文件系统类型> 仅卸除选项中所指定的文件系统。
* -v 执行时显示详细的信息。
* -V 显示版本信息。
* `[文件系统]` 除了直接指定文件系统外，也可以用设备名称或挂入点来表示文件系统。

## 实例

```sh
# 通过设备名卸载
umount -v /dev/sda1

# 通过挂载点卸载
umount -v /mnt/mymount/

# 如果设备正忙，卸载即告失败。
# 卸载失败的常见原因是，某个打开的shell当前目录为挂载点里的某个目录
umount: /mnt/mymount: device is busy
```

参考：

[Linux umount命令](https://www.runoob.com/linux/linux-comm-umount.html)
