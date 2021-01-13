---
title: MySQL GRANT命令-用户授权
date: 2021-01-13 15:20:34
tags:
- Linux
- MySql
- CentOS7
categories: MySql
---

## GRANT语法

在 MySQL 中，拥有 GRANT 权限的用户才可以执行 GRANT 语句，其语法格式如下：

```sh
GRANT priv_type [(column_list)] ON database.table
TO user [IDENTIFIED BY [PASSWORD] 'password']
[, user[IDENTIFIED BY [PASSWORD] 'password']] ...
[WITH with_option [with_option]...]
```

实例：

```sh
GRANT SELECT,INSERT,DELETE,UPDATE ON *.* TO 'root'@'192.125.30.123' IDENTIFIED BY '123456' WITH GRANT OPTION;
```

* priv_type 参数表示权限类型；
* columns_list 参数表示权限作用于哪些列上，省略该参数时，表示作用于整个表；
* database.table 用于指定权限的级别；
* user 参数表示用户账户，由用户名和主机名构成，格式是“'username'@'hostname'”；
* IDENTIFIED BY 参数用来为用户设置密码；
* password 参数是用户的新密码。

WITH 关键字后面带有一个或多个 with_option 参数。这个参数有 5 个选项，详细介绍如下：

* GRANT OPTION：被授权的用户可以将这些权限赋予给别的用户；
* MAX_QUERIES_PER_HOUR count：设置每个小时可以允许执行 count 次查询；
* MAX_UPDATES_PER_HOUR count：设置每个小时可以允许执行 count 次更新；
* MAX_CONNECTIONS_PER_HOUR count：设置每小时可以建立 count 个连接;
* MAX_USER_CONNECTIONS count：设置单个用户可以同时具有的 count 个连接。

GRANT 语句中可用于指定权限级别的值有以下几类格式：

* *：表示当前数据库中的所有表。
* *.*：表示所有数据库中的所有表。
* db_name.*：表示某个数据库中的所有表，db_name 指定数据库名。
* db_name.tbl_name：表示某个数据库中的某个表或视图，db_name 指定数据库名，tbl_name 指定表名或视图名。
* db_name.routine_name：表示某个数据库中的某个存储过程或函数，routine_name 指定存储过程名或函数名。
* TO 子句：如果权限被授予给一个不存在的用户，MySQL 会自动执行一条 CREATE USER 语句来创建这个用户，但同时必须为该用户设置密码。

## 数据库权限的权限类型

| 权限名称 | 对应user表中的字段 | 说明 |
| :-----| :----: | :----: |
| SELECT  |  Select_priv  |  表示授予用户可以使用 SELECT 语句访问特定数据库中所有表和视图的权限。
| INSERT  |  Insert_priv  |  表示授予用户可以使用 INSERT 语句向特定数据库中所有表添加数据行的权限。
| DELETE  |  Delete_priv  |  表示授予用户可以使用 DELETE 语句删除特定数据库中所有表的数据行的权限。
| UPDATE  |  Update_priv  |  表示授予用户可以使用 UPDATE 语句更新特定数据库中所有数据表的值的权限。
| REFERENCES  |  References_priv  |  表示授予用户可以创建指向特定的数据库中的表外键的权限。
| CREATE  |  Create_priv  |  表示授权用户可以使用 CREATE TABLE 语句在特定数据库中创建新表的权限。
| ALTER  |  Alter_priv  |  表示授予用户可以使用 ALTER TABLE 语句修改特定数据库中所有数据表的权限。
| SHOW VIEW	  |  Show_view_priv  |  表示授予用户可以查看特定数据库中已有视图的视图定义的权限。
| CREATE ROUTINE  |  Create_routine_priv  |  表示授予用户可以为特定的数据库创建存储过程和存储函数的权限。
| ALTER ROUTINE  |  Alter_routine_priv  |  表示授予用户可以更新和删除数据库中已有的存储过程和存储函数的权限。
| INDEX  |  Index_priv  |  表示授予用户可以在特定数据库中的所有数据表上定义和删除索引的权限。
| DROP  |  Drop_priv  |  表示授予用户可以删除特定数据库中所有表和视图的权限。
| CREATE TEMPORARY TABLES  |  Create_tmp_table_priv  |  表示授予用户可以在特定数据库中创建临时表的权限。
| CREATE VIEW  |  Create_view_priv  |  表示授予用户可以在特定数据库中创建新的视图的权限。
| EXECUTE ROUTINE  |  Execute_priv  |  表示授予用户可以调用特定数据库的存储过程和存储函数的权限。
| LOCK TABLES  |  Lock_tables_priv  |  表示授予用户可以锁定特定数据库的已有数据表的权限。
| ALL 或 ALL PRIVILEGES 或 SUPER  |  Super_priv  |  表示以上所有权限/超级权限

## 表权限的权限类型

| 权限名称 | 对应user表中的字段 | 说明 |
| :-----| :----: | :----: |
|  SELECT	  |  Select_priv	  |  授予用户可以使用 SELECT 语句进行访问特定表的权限
|  INSERT	  |  Insert_priv	  |  授予用户可以使用 INSERT 语句向一个特定表中添加数据行的权限
|  DELETE	  |  Delete_priv	  |  授予用户可以使用 DELETE 语句从一个特定表中删除数据行的权限
|  DROP	  |  Drop_priv	  |  授予用户可以删除数据表的权限
|  UPDATE	  |  Update_priv	  |  授予用户可以使用 UPDATE 语句更新特定数据表的权限
|  ALTER	  |  Alter_priv 	  |  授予用户可以使用 ALTER TABLE 语句修改数据表的权限
|  REFERENCES	  |  References_priv	  |  授予用户可以创建一个外键来参照特定数据表的权限
|  CREATE	  |  Create_priv	  |  授予用户可以使用特定的名字创建一个数据表的权限
|  INDEX	  |  Index_priv	  |  授予用户可以在表上定义索引的权限
|  ALL 或 ALL PRIVILEGES 或 SUPER	  |  Super_priv	  |  所有的权限名

## 列权限的权限类型

值只能指定为 SELECT、INSERT 和 UPDATE，同时权限的后面需要加上 列名 列表 column-list

## 用户权限类型

```sh
# 创建新的用户，并授予 数据有查询、插入权限，并授予 GRANT 权限
GRANT SELECT,INSERT SLAVE ON *.*  TO  'slave'@'localhost' identified by '123456' WITH GRANT OPTION;

# 查询用户权限
SHOW GRANTS FOR 'slave'@'localhost';
```