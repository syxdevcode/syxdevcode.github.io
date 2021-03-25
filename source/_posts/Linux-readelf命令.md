---
title: Linux readelf命令
date: 2021-03-24 15:01:06
tags:
- Linux
- CentOS7
- ELF
- Linux基础命令
categories:
- Linux基础命令
---

readelf命令用来显示一个或者多个elf格式的目标文件的信息，可以通过它的选项来控制显示哪些信息。这里的elf-file(s)就表示那些被检查的文件。可以支持32位，64位的elf格式文件，也支持包含elf文件的文档（这里一般指的是使用ar命令将一些elf文件打包之后生成的例如lib*.a之类的“静态库”文件）。 

readelf命令，一般用于查看ELF格式的文件信息，常见的文件如在Linux上的可执行文件，动态库(*.so)或者静态库(*.a) 等包含ELF格式的文件。以下命令的使用是基于android编译出来的so文件上面去运行。
















参考：

[readelf命令](https://man.linuxde.net/readelf)