---
title: Linux grep命令
date: 2018-05-10 10:45:46
tags:
- Linux
- Linux基础命令
categories:
- Linux基础命令
---
# Linux grep命令

## 作用

Linux系统中grep命令是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹 配的行打印出来。grep全称是Global Regular Expression Print，表示全局正则表达式版本，它的使用权限是所有用户。

## 格式

grep [options]

## 主要参数

** [options]主要参数：**

－c：只输出匹配行的计数。
－I：不区分大 小写(只适用于单字符)。
－h：查询多文件时不显示文件名。
－l：查询多文件时只输出包含匹配字符的文件名。
－n：显示匹配行及 行号。
－s：不显示不存在或无匹配文本的错误信息。
－v：显示不包含匹配文本的所有行。

** pattern正则表达式主要参数：**

\： 忽略正则表达式中特殊字符的原有含义。
^：匹配正则表达式的开始行。
$: 匹配正则表达式的结束行。
\<：从匹配正则表达 式的行开始。
\\>：到匹配正则表达式的行结束。
[ ]：单个字符，如[A]即A符合要求 。
[ - ]：范围，如[A-Z]，即A、B、C一直到Z都符合要求 。
。：所有的单个字符。
\* ：有字符，长度可以为0。

## grep命令使用简单实例

`ls |grep '1' t*`
:显示所有以t开头的文件中包含‘1’的行

`grep ‘test’ aa bb cc`
:显示在aa，bb，cc文件中匹配test的行。

`grep ‘[a-z]\{5\}’ aa`
:显示所有包含每个字符串至少有5个连续小写字符的字符串的行。

## grep命令使用复杂实例

假设您正在’/usr/src/Linux/Doc’目录下搜索带字符 串’magic’的文件：

``` linux
$ grep magic /usr/src/Linux/Doc/*
sysrq.txt:* How do I enable the magic SysRQ key?
sysrq.txt:* How do I use the magic SysRQ key?
```

其中文件’sysrp.txt’包含该字符串，讨论的是 SysRQ 的功能。
默认情况下，’grep’只搜索当前目录。如果 此目录下有许多子目录，’grep’会以如下形式列出：
grep: sound: Is a directory
这可能会使’grep’ 的输出难于阅读。这里有两种解决的办法：

明确要求搜索子目录：`grep -r`
或忽略子目录：`grep -d skip`

如果有很多 输出时，您可以通过管道将其转到’less’上阅读：
`$ grep magic /usr/src/Linux/Documentation/* | less`
这样，您就可以更方便地阅读。

参考：

[http://www.cnblogs.com/end/archive/2012/02/21/2360965.html](http://www.cnblogs.com/end/archive/2012/02/21/2360965.html)