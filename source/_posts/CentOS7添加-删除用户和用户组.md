---
title: CentOS7添加/删除用户和用户组
date: 2018-04-28 11:43:40
tags:
- Linux
- CentOS7
- Linux基础命令
categories: 
- CentOS7
---
# CentOS7添加/删除用户和用户组

## 新建用户

注：root下操作

``` linux
adduser testuser #新建testuser 用户
passwd testuser  #设置密码
```

## 授权

新创建的用户并不能使用sudo命令，需要给他添加授权。

sudo命令的授权管理是在sudoers文件里的。可以看看sudoers

``` linux
[root@localhost Desktop]# sudoers
bash: sudoers: 未找到命令...
[root@localhost Desktop]# whereis sudoers
sudoers: /etc/sudoers /etc/sudoers.d /usr/libexec/sudoers.so /usr/share/man/man5/sudoers.5.gz
```

找到这个文件位置之后再查看权限：

``` linux
[root@localhost Desktop]# ls -l /etc/sudoers
-r--r-----. 1 root root 4000 3月   6 2015 /etc/sudoers
```

只有只读的权限，如果想要修改的话，需要先添加w权限：

``` linux
[root@localhost Desktop]# chmod -v u+w /etc/sudoers
mode of "/etc/sudoers" changed from 0440 (r--r-----) to 0640 (rw-r-----)
```

然后就可以添加内容了，在下面的一行下追加新增的用户：

``` linux
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
toor    ALL=(ALL)       ALL #新增的用户
```

wq保存退出，这时候要记得将写权限收回：

``` linux
[root@localhost Desktop]# chmod -v u-w /etc/sudoers
mode of "/etc/sudoers" changed from 0640 (rw-r-----) to 0440 (r--r-----)
```

这时候使用新用户登录，使用sudo

## 新建工作组

``` linux
groupadd testgroup
```

## 新建用户同时增加工作组

``` linux
useradd -g testgroup testuser
```

注：：-g 所属组

## 给已有的用户增加工作组

``` linux
usermod -G groupname username
```

## 永久删除用户帐号

``` linux
userdel testuser
groupdel testgroup
usermod -G testgroup testuser #强制删除该用户的主目录和主目录下的所有文件和子目录
```

用户列表文件：`/etc/passwd`
用户组列表文件：`/etc/group`

查看系统中有哪些用户：`cut -d : -f 1 /etc/passwd`
查看可以登录系统的用户：`cat /etc/passwd | grep -v /sbin/nologin | cut -d : -f 1`
查看用户操作：w命令(需要root权限)
查看某一用户：w 用户名
查看登录用户：`who`
查看用户登录历史记录：`last`

参考：

[https://www.linuxidc.com/Linux/2016-11/137549.htm](https://www.linuxidc.com/Linux/2016-11/137549.htm)

[http://www.jb51.net/article/123444.htm](http://www.jb51.net/article/123444.htm)

[http://blog.51cto.com/risingair/1861641](http://blog.51cto.com/risingair/1861641)