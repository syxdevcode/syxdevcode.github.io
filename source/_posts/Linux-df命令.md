---
title: Linux df命令
date: 2020-08-27 13:50:22
tags:
  - Linux
  - Linux基础命令
categories:
  - Linux基础命令
---

## 简介

Linux df 命令用于显示目前在 Linux 系统上的文件系统的磁盘使用情况统计。默认显示单位为 KB。可以利用该命令来获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

## 语法

```sh
df [选项]... [FILE]...
```

选项

- -a 或--all：包含全部的文件系统；
- --block-size=<区块大小>：以指定的区块大小来显示区块数目；
- -h 或--human-readable：以可读性较高的方式来显示信息；
- -H 或--si：与-h 参数相同，但在计算时是以 1000 Bytes 为换算单位而非 1024 Bytes；
- -i 或--inodes：显示 inode 的信息；
- -k 或--kilobytes：指定区块大小为 1024 字节；
- -l 或--local：仅显示本地端的文件系统；
- -m 或--megabytes：指定区块大小为 1048576 字节；
- --no-sync：在取得磁盘使用信息前，不要执行 sync 指令，此为预设值；
- -P 或--portability：使用 POSIX 的输出格式；
- --sync：在取得磁盘使用信息前，先执行 sync 指令；
- -t<文件系统类型>或--type=<文件系统类型>：仅显示指定文件系统类型的磁盘信息；
- -T 或--print-type：显示文件系统的类型；
- -x<文件系统类型>或--exclude-type=<文件系统类型>：不要显示指定文件系统类型的磁盘信息；
- --help：显示帮助；
- --version：显示版本信息。

## 实例

```sh
# 查看系统磁盘设备，默认是KB为单位
df

# 查看磁盘空间占用情况
df -h
```

参考：

[Linux df 命令](https://man.linuxde.net/df)
