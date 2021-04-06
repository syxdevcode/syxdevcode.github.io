---
title: Linux dd命令
date: 2021-04-06 09:53:05
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 简介

dd，是 `device driver` 的缩写，它可以称得上是 "Linux 世界中的搬运工"，它用来读取设备、文件中的内容，并原封不动地复制到指定位置。

使用dd对磁盘操作时，最好使用块设备文件。

## 参数说明

* if=文件名：输入文件名，默认为标准输入。即指定源文件。
* of=文件名：输出文件名，默认为标准输出。即指定目的文件。
* ibs=N：一次读入N个字节，即指定一个块大小为N个字节,默认为 512 字节。
  obs=N：一次输出N个字节，即指定一个块大小为N个字节,默认为 512 字节。
  bs=N：同时设置读入/输出的块大小为bytes个字节。
* cbs=bytes：一次转换bytes个字节，即指定转换缓冲区大小。
* skip=blocks：从输入文件开头跳过blocks个块后再开始复制。
* seek=blocks：从输出文件开头跳过blocks个块后再开始复制。
* count=blocks：仅拷贝blocks个块，块大小等于ibs指定的字节数。
* conv=<关键字>，关键字可以有以下11种：
 conversion：用指定的参数转换文件。
 ascii：转换 ebcdi c为 ascii
 ebcdic：转换 ascii 为 ebcdic
 ibm：转换 ascii 为 alternate ebcdic
 block：把每一行转换为长度为cbs，不足部分用空格填充
 unblock：使每一行的长度都为cbs，不足部分用空格填充
 lcase：把大写字符转换为小写字符
 ucase：把小写字符转换为大写字符
 swap：交换输入的每对字节
 noerror：出错时不停止
 notrunc：不截短输出文件
 sync：将每个输入块填充到ibs个字节，不足部分用空（NUL）字符补齐。
* --help：显示帮助信息
* --version：显示版本信息

注： 块大小可以使用的计量单位表


| 单元大小    | 代码 |  
| :----- | :---- |  
|字节（1B）	|c |
|字节（2B）|	w|
|块（512B）|	b|
|千字节（1024B）  |	k|
|兆字节（1024KB） | M|
|吉字节（1024MB） |	G|

## 实例

```sh
# 制作启动盘
dd if=boot.img of=/dev/fd0 bs=1440k

# 备份磁盘
dd if=/dev/sda of=/root/sda.img
# 恢复磁盘
dd if=/root/sda.img of=/dev/sdb

# 将testfile文件中的所有英文字母转换为大写，然后转成为testfile_1文件：
dd if=testfile_2 of=testfile_1 conv=ucase 


# 备份时进行压缩
dd if=/dev/sda | gzip > /root/sda.img.gz
# 解压
bzip2 -dc /root/sda.img.gz | dd of=/dev/sdc
```

## 其它

* /dev/null，也叫空设备，小名 "无底洞"。任何写入它的数据都会被无情抛弃。
* /dev/zero，可以产生连续不断的 null 的流（二进制的零流），用于向设备或文件写入 null 数据，一般用它来对设备或文件进行初始化。
* /dev/urandom 进行格式化

```sh
# 向磁盘上写一个大文件
dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file

# 从磁盘上读取一个大文件
dd if=/root/1Gb.file bs=64k | dd of=/dev/null

# 配合 time 命令，可以看出不同的块大小数据的写入时间，
# 可以测算出到底块大小为多少时可以实现最佳的写入性能。
time dd if=/dev/zero bs=1024 count=1000000 of=/root/1Gb.file
time dd if=/dev/zero bs=2048 count=500000 of=/root/1Gb.file
time dd if=/dev/zero bs=4096 count=250000 of=/root/1Gb.file
time dd if=/dev/zero bs=8192 count=125000 of=/root/1Gb.file

# 产生随机数据，写到磁盘上
dd if=/dev/urandom of=/dev/sda
```

参考：

[Linux dd 命令](https://www.runoob.com/linux/linux-comm-dd.html)

[dd命令](https://man.linuxde.net/dd)

[dd命令_Linux dd命令：复制（拷贝）文件，并对原文件进行转换](http://c.biancheng.net/linux/dd.html)