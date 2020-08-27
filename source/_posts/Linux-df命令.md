---
title: Linux df命令
date: 2020-08-27 13:50:22
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 简介

Linux df命令用于显示目前在Linux系统上的文件系统的磁盘使用情况统计。默认显示单位为KB。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

## 语法

```sh
df [选项]... [FILE]...
```

选项

* -a或--all：包含全部的文件系统；
* --block-size=<区块大小>：以指定的区块大小来显示区块数目；
* -h或--human-readable：以可读性较高的方式来显示信息；
* -H或--si：与-h参数相同，但在计算时是以1000 Bytes为换算单位而非1024 Bytes；
* -i或--inodes：显示inode的信息；
* -k或--kilobytes：指定区块大小为1024字节；
* -l或--local：仅显示本地端的文件系统；
* -m或--megabytes：指定区块大小为1048576字节；
* --no-sync：在取得磁盘使用信息前，不要执行sync指令，此为预设值；
* -P或--portability：使用POSIX的输出格式；
* --sync：在取得磁盘使用信息前，先执行sync指令；
* -t<文件系统类型>或--type=<文件系统类型>：仅显示指定文件系统类型的磁盘信息；
* -T或--print-type：显示文件系统的类型；
* -x<文件系统类型>或--exclude-type=<文件系统类型>：不要显示指定文件系统类型的磁盘信息；
* --help：显示帮助；
* --version：显示版本信息。

## 实例

```sh
# 查看系统磁盘设备，默认是KB为单位
df 
```

参考：

[Linux df命令](https://man.linuxde.net/df)
