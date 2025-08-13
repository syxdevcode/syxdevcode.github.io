---
title: Linux tail命令
date: 2021-03-29 15:45:29
tags:
  - Linux
  - CentOS7
  - Linux优化
  - Linux基础命令
categories:
  - Linux基础命令
---

tail 命令和 head 命令正好相反，用来查看文件末尾的数据。

基本格式如下：`tail [选项] 文件名`

- -n 行数 K：该选项表示输出最后 K 行，在此基础上，如果使用 `-n +K`，则表示从文件的第 K 行开始输出。
- -c 行数 K: 该选项表示输出文件最后 K 个字节的内容，在此基础上，使用 `-c +K` 则表示从文件第 K 个字节开始输出。
- -f :输出文件变化后新增加的数据。
  <!--more-->
  实例：

```sh
# 查看 /etc/passwd 文件最后 3 行的数据内容
tail -n 3 /etc/passwd
tail -3 /etc/passwd

# 从0行开始查看
docker exec lims-server tail -n +0 AdminNETConfig.json

# 查看 /etc/passwd 文件末尾 100 个字节的数据内容
tail -c 100 /etc/passwd
```

参考：

[Linux tail 命令：显示文件结尾的内容](http://c.biancheng.net/view/737.html)
