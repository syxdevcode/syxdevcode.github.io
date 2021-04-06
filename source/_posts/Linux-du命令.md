---
title: Linux du命令
date: 2021-04-06 13:30:14
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

Linux du（英文全拼：disk usage）命令用于显示目录或文件的大小。du 侧重在文件夹和文件的磁盘占用方面，而 df 则侧重在文件系统级别的磁盘占用方面。

du 会显示指定的目录或文件所占用的磁盘空间。

du 展示的是磁盘空间占用量。ls 展示的是文件内容的大小。

语法

```sh
du [-abcDhHklmsSx][-L <符号连接>][-X <文件>][--block-size]
[--exclude=<目录或文件>][--max-depth=<目录层数>][--help][--version][目录或文件]
```

参数说明：

* -a或-all 显示目录中个别文件的大小。
* -b或-bytes 显示目录或文件大小时，以byte为单位。
* -c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
* -D或--dereference-args 显示指定符号连接的源文件大小。
* -h或--human-readable 以K，M，G为单位，提高信息的可读性。
* -H或--si 与-h参数相同，但是K，M，G是以1000为换算单位。
* -k或--kilobytes 以1024 bytes为单位。
* -l或--count-links 重复计算硬件连接的文件。
* -L<符号连接>或--dereference<符号连接> 显示选项中所指定符号连接的源文件大小。
* -m或--megabytes 以1MB为单位。
* -s或--summarize 仅显示总计。
* -S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小。
* -x或--one-file-xystem 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
* -X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件。
* --exclude=<目录或文件> 略过指定的目录或文件。
* --max-depth=<目录层数> 超过指定层数的目录后，予以忽略。
* --help 显示帮助。
* --version 显示版本信息。

## 单位名称

如果你通过 --block-size 选项设置了块大小，那么，这就会成为你 du 输出信息的单位。
假如上一条没满足，且你设置了环境变量 DU_BLOCK_SIZE，则这会成为你 du 输出信息的单位。
假如上两条都没满足，且你设置了环境变量 BLOCK_SIZE，则这会成为你 du 输出信息的单位。
假如前三条都没满足，且你设置了环境变量 BLOCKSIZE，则这会成为你 du 输出信息的单位。
假如前四条都没满足，且你开启了环境变量 POSIXLY_CORRECT，则 du 输出信息的单位会是 512 bytes。
假如前面的五条都没满足，那么 du 的输出信息的单位就是 1024 bytes，也就是 KB。

## 实例

1，显示目录或者文件所占空间:

```sh
du
608     ./test6
308     ./test4
4       ./scf/lib
4       ./scf/service/deploy/product
4       ./scf/service/deploy/info
12      ./scf/service/deploy
16      ./scf/service
4       ./scf/doc
4       ./scf/bin
32      ./scf
8       ./test3
1288    .
```
只显示当前目录下面的子目录的目录大小和当前目录的总的大小，最下面的1288为当前目录的总大小

2，显示指定文件所占空间

```sh
# du log2012.log 
300     log2012.log
```

3，方便阅读的格式显示test目录所占空间情况：

```sh
# du -h test
608K    test/test6
308K    test/test4
4.0K    test/scf/lib
4.0K    test/scf/service/deploy/product
4.0K    test/scf/service/deploy/info
12K     test/scf/service/deploy
16K     test/scf/service
4.0K    test/scf/doc
4.0K    test/scf/bin
32K     test/scf
8.0K    test/test3
1.3M    test
```

4，排除隐藏文件和隐藏文件夹

```sh
# 了几个隐藏的文件和文件夹
[roc@roclinux ruanjian]$ du -ah .
6.8M    ./wordpress-4.4.1.tar.gz
3.4M    ./curl-7.34.0.tar.gz
980K    ./soft/redis-2.6.16.tar.gz
40M     ./soft/go1.1.2.Linux-amd64.tar.gz
120K    ./soft/.abc
0       ./.bbc/ddd
0       ./.bbc/.ccc
51M     .
 
# 用--exclude的一个很简单的正则匹配
[roc@roclinux ruanjian]$ du -ah --exclude="*/.*" .
6.8M    ./wordpress-4.4.1.tar.gz
3.4M    ./curl-7.34.0.tar.gz
980K    ./soft/redis-2.6.16.tar.gz
40M     ./soft/go1.1.2.Linux-amd64.tar.gz
41M     ./soft
51M     .
```

5，查看磁盘空间

```sh
# 当前文件夹下第一级的大小排序
du -sh *|sort -nr

# 当前文件夹和其子文件夹下的大排序
du -ah .|sort -hr
```

sort -h选项和-n选项的区别：
* -n选项，按数值进行比较，只会傻傻地比较数字，它会认为 98 K大于 2G。
* -h选项，会更加聪明，先优先比较单位（G>M>K），然后再对数值进行比较。


参考：

[Linux du 命令](https://www.runoob.com/linux/linux-comm-du.html)

[du命令_Linux du命令：查看文件夹和文件的磁盘占用情况](http://c.biancheng.net/linux/du.html)

