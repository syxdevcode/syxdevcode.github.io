---
title: Centos7make编译安装
date: 2020-05-10 13:07:36
tags:
- Linux
- CentOS7
- 安装部署
categories: 
- CentOS7
---

```bash
wget http://ftp.gnu.org/gnu/make/make-4.3.tar.gz
tar -zxvf make-4.3.tar.gz
cd make-4.3
./configure
make
make install
ln -s -f /usr/local/bin/make  /usr/bin/make
```

参考：

[Centos7make编译安装](https://www.jianshu.com/p/7ccfe99cab05)
