---
title: Linux shell中的竖线（|）——管道符号
date: 2018-05-10 10:18:29
tags:
- Linux
- Linux基础命令
categories:
- Linux基础命令
---
# Linux shell中的竖线（|）——管道符号

在Linux系统中管道线是由竖杠（|）隔开的若干个命令组成的序列，在管道线中，每个命令运行时都有一个独立的进程。前一个命令的输出正是下一个命令的输入。而管道线中有一类命令也称作“过滤器”，过滤器首先读取输入，然后将输入以某种简单方式进行变换（相当于过滤），再将处理结果输出，例如grep、tail、sort和wc等命令就称为过滤器。

一个管道线中可以包括多条命令。

用法: `command 1 | command 2`

他的功能是把第一个命令command 1执行的结果作为command 2的输入传给command 2

例如：

``` linux
ls -s|sort -nr
```

-s 是file size，-n是numeric-sort，-r是reverse，反转

该命令列出当前目录中的文档(含size)，并把输出送给sort命令作为输入，sort命令按数字递减的顺序把ls的输出排序。

``` linux
ls -s|sort -n
```

按从小到大的顺序输出。

参考：

[http://blog.sina.com.cn/s/blog_6d09b5750100vley.html](http://blog.sina.com.cn/s/blog_6d09b5750100vley.html)

[https://blog.csdn.net/kaosini/article/details/6578115](https://blog.csdn.net/kaosini/article/details/6578115)