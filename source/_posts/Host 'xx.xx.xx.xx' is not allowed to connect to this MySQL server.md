---
title: Host  'xx.xx.xx.xx' is not allowed to connect to this MySQL server
date: 2017-02-17 14:33:37
tags: 
- MySql
categories: MySql
---
# Host  'xx.xx.xx.xx' is not allowed to connect to this MySQL server

注：需要在服务器登录mysql终端

### 改表法

```linux
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> update user set host = '%' where user = 'root';//需要远程连接的用户
mysql> select host,user from user;
+-----------+-----------+
| host      | user      |
+-----------+-----------+
| %         | root      |
| localhost | mysql     |
| localhost | mysql.sys |
+-----------+-----------+
3 rows in set (0.00 sec)
```

### 授权法

```linux
mysql> GRANT ALL PRIVILEGES ON *.* TO 'mysql'@'%' IDENTIFIED BY 'mypassword' WITH GRANT OPTION; //mysql 密码强度有验证；mypassword：密码
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> FLUSH   PRIVILEGES; // 刷新
Query OK, 0 rows affected (0.01 sec)
```

如果你想允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器，并使用mypassword作为密码

```linux
GRANT ALL PRIVILEGES ON *.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
FLUSH   PRIVILEGES;
```

如果你想允许用户myuser从ip为192.168.1.6的主机连接到mysql服务器的dk数据库，并使用mypassword作为密码

```linux
GRANT ALL PRIVILEGES ON dk.* TO 'myuser'@'192.168.1.3' IDENTIFIED BY 'mypassword' WITH GRANT OPTION;
FLUSH   PRIVILEGES;
```