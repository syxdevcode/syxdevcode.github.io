---
title: Linux rcp命令
date: 2020-08-20 17:20:36
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- Linux基础命令
---
## 简介

Linux rcp命令用于复制远程文件或目录。

rcp指令用在远端复制文件或目录，如同时指定两个以上的文件或目录，且最后的目的地是一个已经存在的目录，则它会把前面指定的所有文件或目录复制到该目录中。

## 语法

```sh
rcp [-pr][源文件或目录][目标文件或目录]
rcp [-pr][源文件或目录...][目标文件]
```

参数：

```sh
-p： 保留源文件或目录的属性，包括拥有者，所属群组，权限与时间。
-r： 递归处理，将指定目录下的文件与子目录一并处理。
```

**实例**

使用rcp指令复制远程文件到本地进行保存。

设本地主机当前账户为rootlocal，远程主机账户为root，要将远程主机（192.125.30.5）主目录下的文件"testfile" 复制到本地目录 "test" 中，则输入如下命令：

```sh
# 复制远程文件到本地
rcp root@192.125.30.5:./testfile testfile  
rcp root@192.125.30.5:home/rootlocal/testfile testfile

# 要求当前登录账户cmd 登录到远程主机  
rcp 192.125.30.5:./testfile testfile
```

注意：指令 `rcp` 执行以后不会有返回信息，仅需要在目录 "test" 下查看是否存在文件 "testfile"。
若存在，则表示远程复制操作成功，否则远程复制操作失败。

参考：

[Linux rcp命令](https://www.runoob.com/linux/linux-comm-rcp.html)