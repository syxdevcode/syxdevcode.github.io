---
title: Linux wc命令
date: 2021-03-29 10:01:16
tags:
- Linux
- CentOS7
- Linux优化
- Linux基础命令
categories:
- Linux基础命令
---

Linux wc 命令用于计算字数。利用 `wc` 指令可以计算文件的Byte数、字数、或是列数，若不指定文件名称、或是所给予的文件名为"-"，则 `wc` 指令会从标准输入设备读取数据。

语法 `wc [-clw][--help][--version][文件...]`

参数：

* -c或--bytes或--chars 只显示Bytes数。
* -l或--lines 显示行数。
* -w或--words 只显示字数。
* --help 在线帮助。
* --version 显示版本信息。

实例

在默认的情况下，wc将计算指定文件的 行数、字数，以及字节数。使用的命令为：

```sh
# testfile文件的统计信息
wc testfile
3 92 598 testfile  # testfile文件的行数为3、单词数92、字节数598 

#统计三个文件的信息
wc testfile testfile_1 testfile_2
3 92 598 testfile                    #第一个文件行数为3、单词数92、字节数598  
9 18 78 testfile_1                   #第二个文件的行数为9、单词数18、字节数78  
3 6 32 testfile_2                    #第三个文件的行数为3、单词数6、字节数32  
15 116 708 总用量                    #三个文件总共的行数为15、单词数116、字节数708 

# 显示行数
wc -l
```