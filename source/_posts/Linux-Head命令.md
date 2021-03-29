---
title: Linux Head命令
date: 2021-03-29 09:56:55
tags:
- Linux
- CentOS7
- Linux优化
- Linux基础命令
categories:
- Linux基础命令
---

`head` 命令可用于查看文件的开头部分的内容，有一个常用的参数 `-n` 用于显示行数，默认为 `10`，即显示 `10` 行的内容。

命令格式：`head [参数] [文件]`

参数：

* -q 隐藏文件名
* -v 显示文件名
* -c<数目> 显示的字节数。
* -n<行数> 显示的行数。

实例：

```sh
head -n 15 1.log
```