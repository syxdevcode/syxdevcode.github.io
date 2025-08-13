---
title: CentOS7执行Shell脚本
date: 2020-08-06 14:00:15
tags:
- Linux
- CentOS7
- Shell脚本
categories:
- Shell脚本
---

先用 `chmod` 让Shell脚本有可执行权限，Linux下面用命令如何运行Shell脚本的方法，有两种方法：

```sh
# 1， ./ 加上文件名 .sh
./test.sh

# 2，sh 加上 文件名.sh
sh test.sh
```

如果没有权限，需要先设置权限，命令如下：

```sh
# 添加执行权限
chmod + x testsh

# 转到指定文件的目录下
cd test

#执行文件
./test.sh  
```

参考：

[centos执行.sh文件](https://blog.csdn.net/xiaopu99/article/details/82978743)

[Shell 教程](https://www.runoob.com/linux/linux-shell.html)