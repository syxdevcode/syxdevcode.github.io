---
title: Mysql-CentOS7卸载
date: 2017-02-16 17:05:49
tags: 
- Linux
- MySql
- CentOS7
- 安装部署
categories: MySql
---
# 卸载mysql

命令：

``` linux
[root@localhost usr]# yum remove mysql mysql-server mysql-libs compat-mysql51
[root@localhost usr]# rm -rf /var/lib/mysql
[root@localhost usr]# rm /etc/my.cnf
```

查看安装的rpm包：

```linux
[root@localhost mysql]# rpm -aq | grep -i mysql
MySQL-server-5.6.27-1.el6.x86_64
MySQL-client-5.6.27-1.el6.x86_64
MySQL-devel-5.6.27-1.el6.x86_64
[root@localhost mysql]# rpm -e MySQL-server-5.6.27-1.el6.x86_64
[root@localhost mysql]# rpm -e MySQL-client-5.6.27-1.el6.x86_64
[root@localhost mysql]# rpm -e MySQL-devel-5.6.27-1.el6.x86_64
[root@localhost rc.d]# cd /var/lib/
[root@localhost lib]# rm -rf mysql/
```

```linux
[root@localhost usr]# whereis mysql
mysql: /usr/lib64/mysql
[root@localhost usr]# rm -rf /usr/lib64/mysql
```

卸载自启服务：

```linux
[root@localhost usr]# chkconfig --list | grep -i mysql
[root@localhost usr]# chkconfig --del mysqld
```

参考：

[http://blog.csdn.net/typa01_kk/article/details/49057073](http://blog.csdn.net/typa01_kk/article/details/49057073, "mysql server卸载")