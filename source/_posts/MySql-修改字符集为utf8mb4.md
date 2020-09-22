---
title: MySql 修改字符集为utf8mb4
date: 2020-09-22 09:56:56
tags:
- Linux
- MySql
- CentOS7
- 安装部署
categories: MySql
---

`utf8mb4` 的最低mysql版本支持版本为5.5.3+，若不是，请升级到较新版本。

```sql
mysql> SHOW VARIABLES WHERE Variable_name LIKE 'character_set_%' OR Variable_name LIKE 'collation%';
+--------------------------+----------------------------------+
| Variable_name            | Value                            |
+--------------------------+----------------------------------+
| character_set_client     | utf8                             |
| character_set_connection | utf8                             |
| character_set_database   | utf8                             |
| character_set_filesystem | binary                           |
| character_set_results    | utf8                             |
| character_set_server     | utf8                             |
| character_set_system     | utf8                             |
| character_sets_dir       | /usr/local/mysql/share/charsets/ |
| collation_connection     | utf8_general_ci                  |
| collation_database       | utf8_general_ci                  |
| collation_server         | utf8_general_ci                  |
+--------------------------+----------------------------------+
11 rows in set (0.00 sec)
```

```conf
# 修改database默认的字符集
# 虽然修改了database的字符集为utf8mb4，但是实际只是修改了database新创建的表，默认使用utf8mb4，
# 原来已经存在的表，字符集并没有跟着改变，需要手动为每张表设置字符集
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci

# 修改表默认的字符集 
ALTER TABLE table_name DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

# 修改表默认的字符集和所有字符列的字符集 
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

## 修改MySQL配置文件

```conf
# 对本地的mysql客户端的配置
[client]
default-character-set = utf8mb4

# 对其他远程连接的mysql客户端的配置
[mysql]
default-character-set = utf8mb4

# 本地mysql服务的配置
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

```conf
# 必须保证一下系统变量是 utf8mb4

# 客户端来源数据使用的字符集
character_set_client

# 连接层字符集
character_set_connection

# 当前选中数据库的默认字符集
character_set_database

# 查询结果字符集
character_set_results

# 默认的内部操作字符集
character_set_server
```

参考：

[MySQL中的 utf8 并不是真正的UTF-8编码 ! !](https://blog.csdn.net/qq_39390545/article/details/106946166)

[mysql 修改字符集为utf8mb4](https://www.cnblogs.com/wang666/p/10282755.html)

[更改MySQL数据库的编码为utf8mb4](https://blog.csdn.net/eagle89/article/details/82148751)