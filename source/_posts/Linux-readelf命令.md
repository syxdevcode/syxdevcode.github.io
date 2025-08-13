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

## 命令

&emsp;&emsp;readelf 命令用来显示一个或者多个elf格式的目标文件的信息，可以通过它的选项来控制显示哪些信息。这里的elf-file(s)就表示那些被检查的文件。可以支持32位，64位的elf格式文件，也支持包含elf文件的文档（这里一般指的是使用ar命令将一些elf文件打包之后生成的例如lib*.a之类的 "静态库" 文件）。

&emsp;&emsp;常见的文件如在Linux上的可执行文件，动态库(*.so)或者静态库(*.a) 等包含ELF格式的文件。以下命令的使用是基于android编译出来的so文件上面去运行。

运行 readelf 的时候，除了-v 和 -H 之外，其它的选项必须有一个被指定。 

选项

```sh
-a 
--all 显示全部信息,等价于 -h -l -S -s -r -d -V -A -I. 

-h 
--file-header 显示elf文件开始的文件头信息. 

-l 
--program-headers  
--segments 显示程序头（段头）信息(如果有的话)。 

-S 
--section-headers  
--sections 显示节头信息(如果有的话)。 

-g 
--section-groups 显示节组信息(如果有的话)。 

-t 
--section-details 显示节的详细信息(-S的)。 

-s 
--syms        
--symbols 显示符号表段中的项（如果有的话）。 

-e 
--headers 显示全部头信息，等价于: -h -l -S 

-n 
--notes 显示note段（内核注释）的信息。 

-r 
--relocs 显示可重定位段的信息。 

-u 
--unwind 显示unwind段信息。当前只支持IA64 ELF的unwind段信息。 

-d 
--dynamic 显示动态段的信息。 

-V 
--version-info 显示版本段的信息。 

-A 
--arch-specific 显示CPU构架信息。 

-D 
--use-dynamic 使用动态段中的符号表显示符号，而不是使用符号段。 

-x <number or name> 
--hex-dump=<number or name> 以16进制方式显示指定段内内容。number指定段表中段的索引,或字符串指定文件中的段名。 

-w[liaprmfFsoR] or 
--debug-dump[=line,=info,=abbrev,=pubnames,=aranges,=macro,=frames,=frames-interp,=str,=loc,=Ranges] 显示调试段中指定的内容。 

-I 
--histogram 显示符号的时候，显示bucket list长度的柱状图。 

-v 
--version 显示readelf的版本信息。 

-H 
--help 显示readelf所支持的命令行选项。 

-W 
--wide 宽行输出。 

@file 可以将选项集中到一个文件中，然后使用这个@file选项载入。 
```

## 实例

* 读取可执行文件形式的elf文件头信息：

```sh
readelf -h main 
```

* 读取目标文件形式的elf文件头信息：

```sh
readelf -h myfile.o 
```

* 读取静态库文件形式的elf文件头信息：

```sh
readelf -h libmy.a 
```

* 读取动态库文件形式的elf文件头信息：

```sh
readelf -h libmy.so 
```

* 查看可执行的elf文件程序头表信息：

```sh
readelf -l main 
```

* 查看目标文件的elf文件程序头表信息： 

```sh
readelf -l myfile.o 
```

* 查看静态库文件的elf文件程序头表信息：

```sh
readelf -l libmy.a 
```

* 查看动态库文件的elf文件程序头表信息：

```sh
readelf -l libmy.so 
```

* 查看一个可执行的elf文件的节信息：

```sh
readelf -S main
```

* 查看一个包含调试信息的可执行的elf文件的节信息：

```sh
readelf -S main.debug 
```

* 查看一个目标文件的elf文件的节信息：

```sh
readelf -S myfile.o
```

* 查看一个静态库文件的elf文件的节信息：

```sh
readelf -S libmy.a 
```

* 查看一个动态库文件的elf文件的节信息：

```sh
readelf -S libmy.so 
```

参考：

[readelf命令](https://man.linuxde.net/readelf)