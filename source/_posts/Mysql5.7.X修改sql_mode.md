---
title: Mysql5.7.X修改sql_mode
date: 2020-08-11 14:40:26
tags:
- MySql
- CentOS7
categories: MySql
---

5.7 以及之后的版本中会遇到这样的错误:

```
Expression #2 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'name' which is not functionally dependent on columns in GROUP BY clause;
this is incompatible with sql_mode=only_full_group_by

Expression #1 of ORDER BY clause is not in SELECT list, references column 'sort' which is not in SELECT list; this is incompatible with DISTINCT
```

## 说明

mysql 5.7版本默认的sql配置是:sql_mode="ONLY_FULL_GROUP_BY"，这个配置严格执行了"SQL92标准"。

查看sql_mode的语句:

```sql
select @@GLOBAL.sql_mode;

##输出：
ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

很多从5.6升级到5.7时，为了语法兼容，大部分都会选择调整sql_mode，使其保持跟5.6一致，为了尽量兼容程序。

原因分析：
再输出的结果 `target list` 中，即select后面跟着的字段，还有一个地方 `group by column`，就是 group by后面跟着的字段。由于开启了 `ONLY_FULL_GROUP_BY` 的设置，所以如果一个字段没有在 `target list `
和 `group by` 字段中同时出现，或者是聚合函数的值的话，那么这条sql查询是被mysql认为非法的，会报错误。

## 解决方法

### 暂时修改sql_mode

重启mysql数据库服务之后，ONLY_FULL_GROUP_BY还会出现。

```sql
set @@GLOBAL.sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

### 修改mysql配置文件

修改 mysql 配置文件，`/usr/local/mysql/my.cnf`,在最后一行添加，之后重启mysql服务。

```config
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
```

重启：

```sh
/etc/init.d/mysqld restart
```

参考：

[5分钟学会MySQL-this is incompatible with sql_mode=only_full_group_by错误解决方案](https://blog.csdn.net/qq_42175986/article/details/82384160)

[mysql5.7.x:this is incompatible with DISTINCT](https://blog.csdn.net/feinifi/article/details/54135310)
