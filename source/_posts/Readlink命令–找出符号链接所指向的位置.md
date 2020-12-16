---
title: Readlink命令–找出符号链接所指向的位置
date: 2020-12-15 09:43:56
tags:
- Linux
- MySql
- CentOS7
- Linux基础命令
categories: Linux基础命令
---

Readlink 是 Linux 系统中一个常用工具，主要用来找出符号链接所指向的位置。

语法格式：Readlink [参数] [文件]

常用参数：

-f	递归跟随给出文件名的所有符号链接以标准化,除最后一个外所有组件必须存在
-e	递归跟随给出文件名的所有符号链接以标准化，所有组件都必须存在
-n	不输出尾随的新行
-s	缩减大多数的错误消息
-v	报告所有错误消息
参考实例

查看软链文件真实文件：

```sh
[root@sj0002 local]# readlink mysql
/usr/local/mysql
```

参考：

[Readlink命令](https://www.linuxcool.com/readlink)