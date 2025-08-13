---
title: Linux chown命令
date: 2022-10-26 10:04:49
tags:
- Linux
- Linux基础命令
categories:
- Linux基础命令
---

Linux chown（英文全拼：change owner）命令用于设置文件所有者和文件关联组的命令。

Linux/Unix 是多人多工操作系统，所有的文件皆有拥有者。利用 chown 将指定文件的拥有者改为指定的用户或组，用户可以是用户名或者用户 ID，组可以是组名或者组 ID，文件是以空格分开的要改变权限的文件列表，支持通配符。 。

chown 需要超级用户 root 的权限才能执行此命令。

只有超级用户和属于组的文件所有者才能变更文件关联组。非超级用户如需要设置关联组可能需要使用 chgrp 命令。

## 语法

```sh
chown [-cfhvR] [--help] [--version] user[:group] file...
```

参数 :

user : 新的文件拥有者的使用者 ID
group : 新的文件拥有者的使用者组(group)
-c : 显示更改的部分的信息
-f : 忽略错误信息
-h :修复符号链接
-v : 显示详细的处理信息
-R : 处理指定目录以及其子目录下的所有文件
--help : 显示辅助说明
--version : 显示版本

## 实例

把 /var/run/httpd.pid 的所有者设置 root：

```sh
chown root /var/run/httpd.pid
```

将文件 file1.txt 的拥有者设为 runoob，群体的使用者 runoobgroup :

```sh
chown runoob:runoobgroup file1.txt
```

将当前前目录下的所有文件与子目录的拥有者皆设为 runoob，群体的使用者 runoobgroup:

```sh
chown -R runoob:runoobgroup *
```

把 /home/runoob 的关联组设置为 512 （关联组ID），不改变所有者：

```sh
chown :512 /home/runoob
```

参考：

[Linux chown 命令](https://www.runoob.com/linux/linux-comm-chown.html)