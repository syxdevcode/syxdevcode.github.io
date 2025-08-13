---
title: Mysql执行flush privileges报错：Table 'mysql.servers' doesn't exist
date: 2020-08-19 17:51:40
tags:
---

mysql执行刷新权限报错：`Table 'mysql.servers' doesn't exist`

```sh
mysql> flush privileges;
ERROR 1146 (42S02): Table 'mysql.servers' doesn't exist

mysql> show warnings;
+---------+------+-------------------------------------+
| Level   | Code | Message                             |
+---------+------+-------------------------------------+
| Warning | 1146 | Table 'mysql.servers' doesn't exist |
+---------+------+-------------------------------------+
1 row in set (0.00 sec)

mysql> drop table if exists servers;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CREATE TABLE `servers` (
    -> `Server_name` char(64) NOT NULL,
    -> `Host` char(64) NOT NULL,`Db` char(64) NOT NULL,
    -> `Username` char(64) NOT NULL,
    -> `Password` char(64) NOT NULL,
    -> `Port` int(4) DEFAULT NULL,
    -> `Socket` char(64) DEFAULT NULL,
    -> `Wrapper` char(64) NOT NULL,
    -> `Owner` char(64) NOT NULL,
    -> PRIMARY KEY (`Server_name`)
    -> ) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='MySQL Foreign Servers table';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

SQL脚本：

```sql
CREATE TABLE `servers` (
`Server_name` char(64) NOT NULL,
`Host` char(64) NOT NULL,`Db` char(64) NOT NULL,
`Username` char(64) NOT NULL,
`Password` char(64) NOT NULL,
`Port` int(4) DEFAULT NULL,
`Socket` char(64) DEFAULT NULL,
`Wrapper` char(64) NOT NULL,
`Owner` char(64) NOT NULL,
PRIMARY KEY (`Server_name`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 COMMENT='MySQL Foreign Servers table';
```

参考：

[ERROR 1146 (42S02): Table 'mysql.servers' doesn't exist](https://www.cnblogs.com/ghjbk/p/13495761.html)

[MYSQL ERROR 1146 Table doesnt exist 解析](https://www.jianshu.com/p/8af0b92e4fc8)