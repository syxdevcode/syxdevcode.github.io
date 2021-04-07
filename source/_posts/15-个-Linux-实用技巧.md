---
title: 15 个 Linux 实用技巧
date: 2021-04-06 14:06:36
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

转载：[80% 的人都不会的 15 个 Linux 实用技巧](https://mp.weixin.qq.com/s/VRxp4bwcoLQb1r1vX8wp9w)

## 1. 快速清空文件的方法

```sh
$ > access.log

# 几种最常见的清空文件的方法
: > access.log
true > access.log
cat /dev/null > access.log
echo -n "" > access.log
echo > access.log
truncate -s 0 access.log
```

`:` 在 shell 中是一个内置命令，表示 `no-op`，大概就是空语句的意思，所以 `:` 的那个用法，就是执行命令后，什么都没有输出，将空内容覆盖到文件。

<!--more-->

## 2. 快速生成大文件

```sh
# 生成一个文件名为 file.img 大小为 1G 的文件
dd if=/dev/zero of=file.img bs=1M count=1024
```

## 3. 安全擦除硬盘数据

使用 `/dev/urandom` 生成随机数据，将生成的数据写入 sda 硬盘中，相当于安全的擦除了硬盘数据。

```sh
dd if=/dev/urandom of=/dev/sda
```

## 4. 快速制作系统盘

```sh
dd if=ubuntu-server-amd64.iso of=/dev/sdb
```

## 5. 查看某个进程的运行时间

`ps aux`，通过 `-o` 参数，指定只显示具体的某个字段，会得到更清晰的结果。

```sh
ps -p 15352 -o etimes,etime
ELAPSED     ELAPSED
16665515 192-21:18:35
```

通过 etime 获取该进程的运行时间，可以很直观地看到，进程运行了 192 天
同样，可以通过 -o 指定 rss 可以只获取该进程的内存信息。

```sh
ps -p 15352 -o rss
  RSS
257736
```

## 6. 动态实时查看日志

通过 tail 命令 -f 选项，可以动态地监控日志文件的变化，非常实用

```sh
tail -f test.log
```

如果想在日志中出现 Failed 等信息时立刻停止 tail 监控，可以通过如下命令来实现：

```sh
$ tail -f test.log | sed '/Failed/ q'
```

## 7. 时间戳的快速转换

```sh
# 将时间戳，转换为日期时间
date -d@1617689900 +"%Y-%m-%d %H:%M:%S"

# 查看当前的时间戳
date +%s
1617689900
```

## 8. 优雅的计算程序运行时间

在 Linux 下，可以通过 time 命令，很容易获取程序的运行时间：

```sh
time ./test
real    0m1.003s
user    0m0.000s
sys     0m0.000s
```

可以看到，程序的运行时间为: 1.003s。细心的同学，会看到 real 貌似不等于 user + sys，而且还远远大于。

三个参数的含义：

* real：表示的钟表时间，也就是从程序执行到结束花费的时间；
* user：表示运行期间，cpu 在用户空间所消耗的时间；
* sys：表示运行期间，cpu 在内核空间所消耗的时间；

由于 user 和 sys 只统计 cpu 消耗的时间，程序运行期间会调用 sleep 发生阻塞，也可能会等待网络或磁盘 IO，都会消耗大量时间。因此对于类似情况，real 的值就会大于其它两项之和。

另外，也会遇到 real 远远小于 user + sys 的场景：如果程序在多个 cpu 上并行，那么 user 和 sys 统计时间是多个 cpu 时间，实际消耗时间 real 很可能就比其它两个之和要小。

## 9. 命令行查看ascii码

```sh
man ascii
```

## 10. 优雅的删除乱码的文件

在 Linux 系统中，会经常碰到名称乱码的文件。想要删除它，却无法通过键盘输入名字，有时候复制粘贴乱码名称，终端可能识别不了。

```sh
ls  -i
138957 a.txt  138959 T.txt  132395 ڹ��.txt

find . -inum 132395 -exec rm {} \;
```

命令中，-inum 指定的是文件的 inode 号，它是系统中每个文件对应的唯一编号，find 通过编号找到后，执行删除操作。

## 11. Linux上获取你的公网IP地址

在办公或家庭环境，我们的虚拟机或服务器上配置的通常是内网 IP 地址。

两条命令都可以:

```sh
curl ip.sb
curl ifconfig.me
```

## 12. 如何批量下载网页资源

在 Linux 系统，通过 wget 命令可以轻松下载，而不用写脚本或爬虫

```sh
$ wget -r -nd -np --accept=pdf http://fast.dpdk.org/doc/pdf-guides/
# --accept：选项指定资源类型格式 pdf
```

## 13. 历史命令使用技巧

```sh
!!：重复执行上条命令；
!N：重复执行 history 历史中第 N 条命令，N 可以通过 history 查看；
!pw：重复执行最近一次，以pw开头的历史命令，非常有用；
!$：表示最近一次命令的最后一个参数；
```

```sh
$ vim /root/sniffer/src/main.c
$ mv !$ !$.bak
# 相当于
$ mv /root/sniffer/src/main.c /root/sniffer/src/main.c.bak
```

当前工作目录是 root，想把 main.c 改为 main.c.bak。正常情况你可能需要敲 2 遍包含 main.c 的长参数，当然你也可能会选择直接复制粘贴。通过使用 `!$` 变量，可以很轻松优雅的实现改名。

## 14. 快速搜索历史命令

通过执行 Ctrl + r，然后键入要所搜索的命令关键词，进行搜索，回车就可以执行，非常高效。

## 15. 设置环境变量，排除空格

需要配置环境变量。

```sh
# 忽略记录空格开始的命令
export HISTCONTROL=ignorespace
```

在所要执行的命令前，加一个空格，那这条命令就不会被 history 保存到历史记录。

有时候，执行的命令中包含敏感信息，这个小技巧就显得非常实用了，你也不会再因为忘记执行 `history -c` 而烦恼了。