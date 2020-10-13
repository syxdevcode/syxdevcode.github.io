---
title: mysql中binlog format格式简介
date: 2020-08-27 09:34:41
tags:
- Linux
- MySql
- CentOS7
- 安装部署
categories: MySql
---

## 简介

```sh
# 查看binlog的格式的脚本
show GLOBAL variables like '%binlog_format%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
1 row in set (0.01 sec)
```

mysql复制主要有三种方式：
基于SQL语句的复制(`statement-based replication, SBR`)
基于行的复制(`row-based replication, RBR`)
混合模式复制(`mixed-based replication, MBR`)。

对应的 `binlog` 的格式也有三种：`STATEMENT`，`ROW`，`MIXED`。

### STATEMENT模式（SBR）

每一条会修改数据的sql语句会记录到 `binlog` 中。优点是并不需要记录每一条sql语句和每一行的数据变化，减少了 `binlog` 日志量，节约IO，提高性能。缺点是在某些情况下会导致`master-slave` 中的数据不一致(如 `sleep()` 函数， `last_insert_id()`，以及 `user-defined functions(udf)` 等会出现问题)

### ROW模式（RBR）

不记录每条sql语句的上下文信息，仅需记录哪条数据被修改了，修改成什么样了。而且不会出现某些特定情况下的存储过程、或 `function`、或 `trigger` 的调用和触发无法被正确复制的问题。缺点是会产生大量的日志，尤其是 `alter table` 的时候会让日志暴涨。

### MIXED模式（MBR）

以上两种模式的混合使用，一般的复制使用 `STATEMENT` 模式保存 `binlog` ，对于 `STATEMENT` 模式无法复制的操作使用ROW模式保存 `binlog`，MySQL会根据执行的SQL语句选择日志保存方式。
 
`binlog` 复制配置

在mysql的配置文件my.cnf中，可以通过一下选项配置 `binglog` 相关

```s
# binlog日志格式，mysql默认采用ROW，建议使用mixed
binlog_format=MIXED

# binlog日志文件
log-bin=/usr/local/mysql/binlogs/bin-log

# binlog过期清理时间
expire_logs_days=7

# binlog每个日志文件大小
max_binlog_size=100m

# binlog缓存大小
binlog_cache_size=4m

# 最大binlog缓存大小
max_binlog_cache_size=512m
```

**MIXED说明**

对于执行的SQL语句中包含 `now()` 这样的时间函数，会在日志中产生对应的 `unix_timestamp()*1000` 的时间字符串，`slave` 在完成同步时，取用的是 `sqlEvent` 发生的时间来保证数据的准确性。另外对于一些功能性函数 `slave` 能完成相应的数据同步，而对于上面指定的一些类似于 `UDF` 函数，导致 `Slave`无法知晓的情况，则会采用 `ROW` 格式存储这些 `Binlog` ，以保证产生的 `Binlog` 可以供 `Slave` 完成数据同步。

比较以下 `SBR` 和 `RBR` 两种模式各自的优缺点：

**SBR 的优点：**

* 历史悠久，技术成熟
* `binlog` 文件较小
* `binlog` 中包含了所有数据库更改信息，可以据此来审核数据库的安全等情况
* `binlog` 可以用于实时的还原，而不仅仅用于复制
* 主从版本可以不一样，从服务器版本可以比主服务器版本高


**SBR 的缺点：**

不是所有的 `UPDATE` 语句都能被复制，尤其是包含不确定操作的时候。
调用具有不确定因素的 `UDF` 时复制也可能出问题
使用以下函数的语句也无法被复制：

* LOAD_FILE()
* UUID()
* USER()
* FOUND_ROWS()
* SYSDATE() (除非启动时启用了 `--sysdate-is-now` 选项)

SBR：

* `INSERT ... SELECT` 会产生比 `RBR` 更多的行级锁
* 复制需要进行全表扫描(`WHERE` 语句中没有使用到索引)的 `UPDATE` 时，需要比 `RBR` 请求更多的行级锁
* 对于有 `AUTO_INCREMENT` 字段的 `InnoDB` 表而言，`INSERT` 语句会阻塞其他 `INSERT` 语句
* 对于一些复杂的语句，在从服务器上的耗资源情况会更严重，而 `RBR` 模式下，只会对那个发生变化的记录产生影响
* 存储函数(不是存储过程)在被调用的同时也会执行一次 `NOW()` 函数，这个可以说是坏事也可能是好事
* 确定了的 `UDF` 也需要在从服务器上执行
* 数据表必须几乎和主服务器保持一致才行，否则可能会导致复制出错
* 执行复杂语句如果出错的话，会消耗更多资源

**RBR 的优点：**

任何情况都可以被复制，这对复制来说是最安全可靠的
和其他大多数数据库系统的复制技术一样
多数情况下，从服务器上的表如果有主键的话，复制就会快了很多
复制以下几种语句时的行锁更少：

* INSERT ... SELECT
* 包含 `AUTO_INCREMENT` 字段的 `INSERT`
* 没有附带条件或者并没有修改很多记录的 `UPDATE` 或 `DELETE` 语句
* 执行 `INSERT`，`UPDATE`，`DELETE` 语句时锁更少
* 从服务器上采用多线程来执行复制成为可能

**RBR 的缺点：**

* `binlog` 大了很多
* 复杂的回滚时 `binlog` 中会包含大量的数据
* 主服务器上执行 `UPDATE` 语句时，所有发生变化的记录都会写到 `binlog` 中，而 `SBR` 只会写一次，这会导致频繁发生 `binlog` 的并发写问题
* `UDF` 产生的大 `BLOB` 值会导致复制变慢
* 无法从 `binlog` 中看到都复制了写什么语句
* 当在非事务表上执行一段堆积的SQL语句时，最好采用 `SBR` 模式，否则很容易导致主从服务器的数据不一致情况发生

另外，针对系统库 `mysql` 里面的表发生变化时的处理规则如下：

* 如果是采用 `INSERT`，`UPDATE`，`DELETE` 直接操作表的情况，则日志格式根据 `binlog_format` 的设定而记录
* 如果是采用 `GRANT`，`REVOKE`，`SET PASSWORD` 等管理语句来做的话，那么无论如何都采用 `SBR` 模式记录

注：采用 `RBR` 模式后，能解决很多原先出现的主键重复问题。

参考：

[mysql中binlog_format模式与配置详解](https://www.cnblogs.com/langtianya/p/5504774.html)