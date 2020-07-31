---
title: Mysql-CentOS7单机离线安装
date: 2020-05-13 13:07:42
tags:
- Linux
- MySql
- CentOS7
- 安装部署
categories: MySql
---

## 检查

### 检查mysql是否存在

```sh
rpm -qa | grep mysql
```

### 检查mysql组合用户是否存在

```sh
# 检查mysql组和用户是否存在，如无则创建
cat /etc/group | grep mysql
cat /etc/passwd | grep mysql 
```

查看全部用户：

```sh
cat /etc/passwd|grep -v nologin|grep -v halt|grep -v shutdown|awk -F ":" '{print $1 "|" $3 "1" $4}' | more
```

若不存在，则创建mysql组和用户

```sh
# 创建mysql用户组
groupadd mysql

# 创建一个用户名为mysql的用户，并加入mysql用户组
useradd -g mysql mysql

# 制定mysql用户的password
passwd mysql
```

注：

```sh
userdel peter
groupdel peter
```

参考：[centos系统添加/删除用户和用户组](https://www.cnblogs.com/nyfz/p/8557137.html)

## 下载离线包

官网下载地址：[https://dev.mysql.com/downloads/mysql/5.7.html#downloads](https://dev.mysql.com/downloads/mysql/5.7.html#downloads)

版本选择，可以选择一下两种方式：

```sh
Red Hat Enterprise Linux
Linux - Generic
```

## 上传离线包到服务器

```sh
tar -zxvf mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz
mv mysql-5.7.26-linux-glibc2.12-x86_64 mysql
mv mysql /usr/local
```

## 配置

```sh
# 更改所属的组和用户
cd /usr/local/
chown -R mysql mysql/
chgrp -R mysql mysql/
cd mysql/
mkdir data
chown -R mysql:mysql data

[centos系统添加/删除用户和用户组](https://www.cnblogs.com/nyfz/p/8557137.html)

# 在/etc下创建my.cnf文件

# 进入/usr/local/mysql文件夹下
cd /usr/local/mysql

# 创建my.cnf文件
touch my.cnf #或者cd ''>my.conf

# 编辑my.cnf
vim my.cnf
```

my.cnf:

```sh
[mysql]
socket=/var/lib/mysql/mysql.sock
# set mysql client default chararter
default-character-set=utf8

[mysqld]
socket=/var/lib/mysql/mysql.sock
bind-address=0.0.0.0
# set mysql server port  
port = 3306 #默认是3306，防止端口冲突发生，可以避免使用3306mysql默认端口
# set mysql install base dir
basedir=/usr/local/mysql
# set the data store dir
datadir=/usr/local/mysql/data
# set the number of allow max connnection
max_connections=3000
# set server charactre default encoding
character-set-server=utf8
# the storage engine
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
explicit_defaults_for_timestamp=true

[mysql.server]
user=mysql
basedir=/usr/local/mysql
```

参考：

语法
chown [-cfhvR] [--help] [--version] user[:group] file...
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

## 安装

```sh
# 进入mysql
cd /usr/local/mysql

# 安装mysql
bin/mysql_install_db --user=mysql --basedir=/usr/local/mysql/ --datadir=/usr/local/mysql/data/

cp ./support-files/mysql.server /etc/init.d/mysqld

# 设置文件及目录权限
chmod +x /etc/init.d/mysqld
chown 777 my.cnf
chown -R mysql:mysql data
ls -l
```

## 启动mysql

```sh
/etc/init.d/mysqld restart
```

报错：

```sh
[root@renshelaodongbangong-sj0001 mysql]# /etc/init.d/mysqld restart
MySQL server PID file could not be found!                  [FAILED]
Starting MySQL.Logging to '/usr/local/mysql/data/renshelaodongbangong-sj0001.err'.
                                                           [  OK  ]
```

解决方案：

```sh
#找到是否已经有进程占用
ps aux|grep mysql
kill -9 100684

# 确定是否还有占用
ps aux|grep mysql

# 启动
/etc/init.d/mysqld restart
```

## 开机启动

```sh
chkconfig --level 35 mysqld on
chkconfig --list mysqld
chmod +x /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
chkconfig --list mysqld

#检查状态
service mysqld status
```

## 修改配置

```sh
vim /etc/profile

# 修改/etc/profile，在最后添加如下内容:

#set mysql environment
export PATH=$PATH:/usr/local/mysql/bin

# 使文件生效
[root@ren mysql]# vim /etc/profile
[root@ren mysql]# source /etc/profile
```

## 设置密码

```sh
# 获取初始密码
cat /root/.mysql_secret
ugcNJt;ZAx7+
```

```sh
# 登录
mysql -uroot -p
Enter password: #此处填写上边获取到的初始密码
# 修改
set password=password('password');
UPDATE user SET Password = password('password') WHERE User = 'mysql';
flush privileges
exit

# 验证新密码
mysql -uroot -p
Enter password: #此处输入新密码

show tables;
show databases;
```

如果报错：ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)，
则使用下面的方式连接：

```sh
mysql -uroot -h 127.0.0.1 -p
```

## 添加远程访问权限

```sh
mysql> use mysql
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set host='%' where user='root';
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> select host,user from user;
+-----------+---------------+
| host      | user          |
+-----------+---------------+
| %         | root          |
| localhost | mysql.session |
| localhost | mysql.sys     |
+-----------+---------------+
3 rows in set (0.00 sec)
```

## 重启mysql生效

```sh
/etc/init.d/mysqld restart
```

## 配置防火墙

注意：针对各种云，因为有专门策略，所以不需要操作防火墙；

```sh
firewall-cmd --zone=public --add-port=3306/tcp --permanent
firewall-cmd --reload

## 重新查询端口是否开放
firewall-cmd --query-port=3306/tcp

## 查看监听的端口
netstat -lnpt

#关闭端口
firewall-cmd --zone=public --remove-port=3306/tcp --permanent

## 查看防火墙所有开放的端口
firewall-cmd --zone=public --list-ports
```

## 查看监听的端口

```sh
netstat -lnpt
```

## 执行脚本

```sql
CREATE DATABASE data1

use data1

进入mysql：

-- 执行脚本
source /sql/data1.sql
```

参考：

[centos7下使用mysql离线安装包安装mysql5.7](https://www.cnblogs.com/yy3b2007com/p/10497787.html)
