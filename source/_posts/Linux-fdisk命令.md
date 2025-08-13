---
title: Linux fdisk命令
date: 2020-08-27 17:36:38
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 简介

Linux fdisk是一个创建和维护分区表的程序，它兼容DOS类型的分区表、BSD或者SUN类型的磁盘列表。

## 语法

`fdisk [必要参数] [选择参数]`

**必要参数：**

```sh
-l 列出素所有分区表
-u 与"-l"搭配使用，显示分区数目
```

**选择参数：**

```sh
-s<分区编号> 指定分区
-v 版本信息
```

**菜单操作说明**

```sh
m ：显示菜单和帮助信息
a ：活动分区标记/引导分区
d ：删除分区
l ：显示分区类型
n ：新建分区
p ：显示分区信息
q ：退出不保存
t ：设置分区号
v ：进行分区检查
w ：保存修改
x ：扩展应用，高级功能
```

参考：

[Linux fdisk命令](https://www.runoob.com/linux/linux-comm-fdisk.html)

[fdisk命令](https://man.linuxde.net/fdisk)