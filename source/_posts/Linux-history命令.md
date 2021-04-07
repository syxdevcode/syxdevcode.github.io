---
title: Linux history命令
date: 2021-04-07 09:04:03
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---

## 简介

history命令用于显示指定数目的指令命令，读取历史命令文件中的目录到历史命令缓冲区和将历史命令缓冲区中的目录写入命令文件。

该命令单独使用时，仅显示历史命令，在命令行中，可以使用符号 `!` 执行指定序号的历史命令。例如，要执行第2个历史命令，则输入 `!2`。

历史命令是被保存在内存中的，当退出或者登录shell时，会自动保存或读取。在内存中，历史命令仅能够存储 1000 条历史命令，该数量是由环境变量 `HISTSIZE` 进行控制。

<!--more-->

## 语法

`history(选项)(参数)`

## 选项

* -c：清空当前历史命令；
* -a：将历史命令缓冲区中命令写入历史命令文件中；
* -r：将历史命令文件中的命令读入当前历史命令缓冲区；
* -w：将当前历史命令缓冲区命令写入历史命令文件中。

## 参数

n：打印最近的n条历史命令。

显示最近使用的10条历史命令

```sh
history 10
```

## 实例

### 设置时间戳

```sh
# 设置环境变量
export HISTTIMEFORMAT='%F %T '

# 清空 
echo "" > .bash_history

# 查看历史
cat .bash_history
# 输出：
#1617758318
history -w
```

### 使用叹号定位

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

### 快速搜索历史命令

通过执行 Ctrl + r，然后键入要所搜索的命令关键词，进行搜索，回车就可以执行，非常高效。

### 清除所有命令

彻底删除所有的历史命令:

```sh
history -c
history -w
```

## history的配置

1. 设置历史记录的时间：

```sh
# 注意有个空格, 这样在显示时日期与命令之间会有空格分隔
export HISTTIMEFORMAT='%F %T '
```

2. 控制历史命令记录的总个数：

```sh
# 设置内存中的history命令的个数
export HISTSIZE=1000

# 设置文件中的history命令的个数
export HISTFILESIZE=1000
```

3. 更换历史命令的存储位置：

一般情况下，历史命令会被存储在 `~/.bash_history` 文件中。

如果不想存储在这个文件中，而想存储在其他文件中，那么可以通过下面的方式来更改：

```sh
export HISTFILE=~/history.log
```

4. 个性化的配置：

```sh
# 清除整个命令历史中的重复条目
export HISTCONTROL=erasedups

# 忽略记录命令历史中连续重复的命令
export HISTCONTROL=ignoredups

# 忽略记录空格开始的命令
export HISTCONTROL=ignorespace

# 等价于ignoredups和ignorespace
export HISTCONTROL=ignoreboth
```

## 保护重要命令隐私

### 第一种解决方案：

通过设置 `HISTCONTROL=ignorespace`，可以让 `history` 不记录 命令前加空格 特殊输入。

* 第1步：设置 `HISTCONTROL` 环境变量：`export HISTCONTROL=ignorespace`。
* 第2步：输入重要命令时，记得在输入命令前加上空格。

执行 `history`，可以看到刚输入的重要命令没有出现在 `history` 中。

## 第二种解决方案：

* 第1步：设置 `HISTIGNORE` 环境变量 `export HISTIGNORE=*`。
* 第2步：输入重要命令，比如 `mysql -uroot-p123` 。
* 第3步：查看你的 `history`，可以看到刚输入的 `mysql` 命令没有记录在 `history` 中。
* 第4步：恢复命令的记录 `export HISTIGNORE=`。

第4步后，系统又恢复正常，输入的命令又能被正常记录了。

参考：

[history命令](https://man.linuxde.net/history)

[history命令_Linux history命令：查看和执行历史命令](http://c.biancheng.net/linux/history.html)