---
title: 查看centos已经安装的包
date: 2020-05-10 10:01:47
tags:
- Linux
- CentOS7
- 安装部署
categories: 
- CentOS7
---

1. rpm包安装的，可以用rpm -qa看到，如果要查找某软件包是否安装，用 rpm -qa | grep "软件或者包的名字"。
2. deb包安装的，可以用dpkg -l能看到。如果是查找指定软件包，用dpkg -l | grep "软件或者包的名字"； 
3. yum方法安装的，可以用yum list installed查找，如果是查找指定包，命令后加 | grep "软件名或者包名"； 
4. 如果是以源码包自己编译安装的，例如.tar.gz或者tar.bz2形式的，这个只能看可执行文件是否存在了， 
上面两种方法都看不到这种源码形式安装的包。如果是以root用户安装的，可执行程序通常都在/sbin:/usr/bin目录下

5. pip安装的所有包：

```sh
pip list

rpm -qa | grep libcurl
rpm -qa | grep openssl-libs
rpm -qa | grep krb5-libs
rpm -qa | grep zlib

rpm -qa | grep lttng-ust
rpm -qa | grep libicu

yum list installed | grep lttng-ust

yum list installed | libicu

rpm -qa | grep bzip2
```


lttng-ust
https://lttng.org/docs/v2.11/#doc-fedora

bzip2 --安装
https://www.cnblogs.com/thrillerz/p/3935789.html