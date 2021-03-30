---
title: Linux xargs命令
date: 2021-03-30 16:11:22
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

xargs（英文全拼： eXtended ARGuments）是给命令传递参数的一个过滤器，也是组合多个命令的一个工具。它能够捕获一个命令的输出，然后传递给另外一个命令。

xargs 可以将管道或标准输入（stdin）数据转换成命令行参数，也能够从文件的输出中读取数据。xargs 一般是和管道一起使用。

xargs 默认的命令是 echo，这意味着通过管道传递给 xargs 的输入将会包含换行和空白，不过通过 xargs 的处理，换行和空白将被空格取代。

**命令格式:**

`somecommand |xargs -item  command`

**参数：**

* -a file 从文件中读入作为 stdin
* -e flag ，注意有的时候可能会是-E，flag必须是一个以空格分隔的标志，当xargs分析到含有flag这个标志的时候就停止。
* -p 当每次执行一个argument的时候询问一次用户。
* -n num 后面加次数，xargs生成的命令行参数，每次传递几个参数给其后面的命令执行，默认是用所有的。
* -t 表示先打印命令，然后再执行。
* -i 或者是-I，这得看linux支持了，将xargs的每项名称，一般是一行一行赋值给 {}，可以用 {} 代替。
* -r no-run-if-empty 当xargs的输入为空的时候则停止xargs，不用再去执行了。
* -s num 命令行的最大字符数，指的是 xargs 后面那个命令的最大命令行字符数。
* -L num 从标准输入一次读取 num 行送给 command 命令。
* -l 同 -L。
* -d delim 分隔符，默认的xargs分隔符是回车，argument的分隔符是空格，这里修改的是xargs的分隔符。
* -x exit的意思，主要是配合-s使用。。
* -P 修改最大的进程数，默认是1，为0时候为as many as it can ，这个例子我没有想到，应该平时都用不到的吧。

**实例**

xargs 用作替换工具，读取输入数据重新格式化后输出。

```sh
[root@host-192-125-30-82 test]# cat source.txt | xargs
1123 ls: 无法访问dd: 没有那个文件或目录 ls: 无法访问dd: 没有那个文件或目录
[root@host-192-125-30-82 test]# cat source.txt | xargs -n2
1123 ls:
无法访问dd: 没有那个文件或目录
ls: 无法访问dd:
没有那个文件或目录
```

`-d` 选项可以自定义一个定界符

```sh
echo "nameXnameXnameXname" | xargs -dX
name name name name

echo "nameXnameXnameXname" | xargs -dX -n2
name name
name name
```

读取 stdin，将格式化后的参数传递给命令

假设一个命令为 sk.sh 和一个保存参数的文件 arg.txt：

```sh
#!/bin/bash
#sk.sh命令内容，打印出所有参数。

echo $*
```

arg.txt文件内容：

```sh
# cat arg.txt
aaa
bbb
ccc
```

xargs 的一个选项 -I，使用 -I 指定一个替换字符串 {}，这个字符串在 xargs 扩展时会被替换掉，当 -I 与 xargs 结合使用，每一个参数命令都会被执行一次：

```sh
# cat arg.txt | xargs -I {} ./sk.sh -p {} -l

-p aaa -l
-p bbb -l
-p ccc -l
```

复制所有图片文件到 `/data/images` 目录下：

```sh
ls *.jpg | xargs -n1 -I {} cp {} /data/images
```

**xargs 结合 find 使用**

用 rm 删除太多的文件时候，可能得到一个错误信息：`/bin/rm Argument list too long`. 用 xargs 去避免这个问题：

```sh
find . -type f -name "*.log" -print0 | xargs -0 rm -f
xargs -0 将 \0 作为定界符。
```

统计一个源代码目录中所有 php 文件的行数：

```sh
find . -type f -name "*.php" -print0 | xargs -0 wc -l
```

查找所有的 jpg 文件，并且压缩它们：

```sh
find . -type f -name "*.jpg" -print | xargs tar -czvf images.tar.gz
```

**xargs 其他应用**

假如你有一个文件包含了很多你希望下载的 URL，你能够使用 xargs下载所有链接：

```sh
# cat url-list.txt | xargs wget -c
```

参考：

[Linux xargs 命令](https://www.runoob.com/linux/linux-comm-xargs.html)