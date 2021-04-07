---
title: Linux重定向符号-<,<<,>,>>
date: 2021-03-30 15:22:17
tags:
- Linux
- CentOS7
- Linux优化
- Linux基础命令
- Linux符号
categories:
- Linux基础命令
---

箭头的指向就是数据的流向。

数字说明

* 1、标准输入（英文：stdin）： 代码为 `0`，使用 `<` 或 `<<` 。数据流从右向左。
* 2、标准正常输出（英文：stdout）：代码为 `1`，使用 `>` 或 `>>` 。数据流从左向右。
* 3、标准错误输出（英文：stderr）：代码为 `2` ，使用 `2>` 或 `2>>` 。数据流从左向右。
* &  ：表示等同于的意思，如 `&1`。 
* &>file：将标准输入和标准错误输出到重定向到文件。
<!--more-->
例如：`2>&1` 表示2的输出重定向等同于1，也就是标准错误输出重定向到标准输出。

## 命令 < 文件

将文件作为命令的标准输入

```sh
[root@host-192-125-30-82 test]# cat source.txt
123
[root@host-192-125-30-82 test]# echo '123' > source.txt
[root@host-192-125-30-82 test]# xargs -n 2 < source.txt
123
```

## 命令 << 分界符

从标准输入中读入，直到遇到分界符停止。

```sh
[root@host-192-125-30-82 test]# cat >> source.txt <<EOF
> 23456
> EOF
[root@host-192-125-30-82 test]# cat source.txt
123
23456
```

## 命令 < 文件1 > 文件2	

将文件1作为命令的标准输入并将标准输出到文件2

```sh
[root@host-192-125-30-82 test]# touch source1.txt
[root@host-192-125-30-82 test]# xargs -n 2 <source.txt > source1.txt
[root@host-192-125-30-82 test]# cat source1.txt
123 23456
```

## 命令 > 文件

将标准输出重定向到文件中（清除原有文件中的数据）

```sh
[root@host-192-125-30-82 test]# cat source.txt
123
[root@host-192-125-30-82 test]# echo '456' > source.txt
[root@host-192-125-30-82 test]# cat source.txt
456
```

## 命令 2> 文件

将错误输出重定向到文件中（清除原有文件中的数据）

```sh
# 这里使用错误命令 -2
[root@host-192-125-30-82 test]# xargs -n -2 <source.txt 2> source1.txt
[root@host-192-125-30-82 test]# cat source1.txt
xargs: 选项 -n 的值必须 >= 1
Usage: xargs [OPTION]... COMMAND INITIAL-ARGS...
Run COMMAND with arguments INITIAL-ARGS and more arguments read from input.
```

## 命令 >> 文件

将标准输出重定向到文件中（在原有的内容后追加）

```sh
[root@host-192-125-30-82 test]# cat source.txt
123
[root@host-192-125-30-82 test]# echo '456' >> source.txt
[root@host-192-125-30-82 test]# cat source.txt
123
456
```

## 命令 2>> 文件

将错误输出重定向到文件中（在原有内容后面追加）

```sh
# 这里使用错误命令 -2
[root@host-192-125-30-82 test]# cat source1.txt
456
[root@host-192-125-30-82 test]# xargs -n -2 <source.txt 2>> source1.txt
[root@host-192-125-30-82 test]# cat source1.txt
456
xargs: 选项 -n 的值必须 >= 1
```

## 命令 &>> 文件

等同于：命令 >> 文件 2>&1

将标准输出和错误输出共同写入文件（在原有内容后追加）

```sh
[root@host-192-125-30-82 test]# echo '1123' > source.txt
[root@host-192-125-30-82 test]# cat source.txt
1123
[root@host-192-125-30-82 test]# ls dd >> source.txt 2>&1
[root@host-192-125-30-82 test]# cat source.txt
1123
ls: 无法访问dd: 没有那个文件或目录

[root@host-192-125-30-82 test]# ls dd &>> source.txt
[root@host-192-125-30-82 test]# cat source.txt
1123
ls: 无法访问dd: 没有那个文件或目录
ls: 无法访问dd: 没有那个文件或目录
```

参考：

[linux重定向符——"<"、"<<"、">"和">>"](https://www.cnblogs.com/walkwaters/p/12459727.html)

[Linux中的特殊符号-重定向符号](https://blog.csdn.net/u014360942/article/details/72630322)