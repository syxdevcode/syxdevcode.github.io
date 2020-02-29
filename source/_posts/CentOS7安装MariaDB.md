---
title: CentOS7安装MariaDB
date: 2018-08-14 15:08:05
tags:
- Linux
- MySql
- MariaDB
- CentOS7
- 安装部署
categories: MariaDB
---
# CentOS7安装MariaDB

## MariaDB简介

官网：[https://mariadb.org/](https://mariadb.org/)

MariaDB Github地址：[https://github.com/MariaDB/server](https://github.com/MariaDB/server)

## 安装MariaDB

```bash
su root ## 切换root用户
yum -y install mariadb-server mariadb ## 下载MariaDB
yum -y install mariadb-server mariadb ## 启动MariaDB服务
systemctl enable mariadb ## 设置默认开机启动 可选
```

## 测试是否成功

```bash
[root@localhost toor]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

## 运行一次mysql_secure_installation安全配置向导

```bash
#由于一开始安装MariaDB数据库后, root用户默认密码为空, 所以只需要按Enter键
Enter current password for root (enter for none):
OK, successfully used password, moving on...

#是否设置root用户的新密码
Set root password? [Y/n] y

#录入新密码
New password:

#确认新密码
Re-enter new password:

#是否删除匿名用户,生产环境建议删除
Remove anonymous users? [Y/n] y
... Success!

#是否禁止root远程登录,根据自己的需求选择
Disallow root login remotely? [Y/n] n
... skipping.

#是否删除test数据库
Remove test database and access to it? [Y/n] y

#是否重新加载权限表
Reload privilege tables now? [Y/n] y
 ... Success!
```

## 开放端口

```bash
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload
```

## 开放远程连接

```bash
mysql -u root -p ## 登录mysql xing123123
```

```bash
#任何远程主机都可以访问数据库
mysql> grant all privileges on *.* to 'root'@'%' identified by '123456';

#使修改生效
mysql> flush privileges;
mysql> exit
```

## 修改编码为UTF-8

```bash
vim /etc/my.cnf
```

添加以下内容：

```bash
[mysqld]
character-set-server=utf8
```

需要重启数据库

```bash
## 查看数据库编码

 mysql -uroot -p
show variables like 'character%';
```

结果：

```bash
MariaDB [(none)]> show variables like 'character%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8                       |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8                       |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

## 使用mysql workbench连接

![微信截图_20180905104709.png](/img/微信截图_20180905104709.png)

参考：

[Centos7 安装配置MariaDB](https://blog.csdn.net/toubennuhai/article/details/70808610)