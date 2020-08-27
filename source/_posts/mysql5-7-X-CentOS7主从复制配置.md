---
title: mysql5.7.X-CentOS7主从复制配置
date: 2020-08-19 11:12:04
tags:
- Linux
- MySql
- CentOS7
- 安装部署
categories: MySql
---
## 简介

主从复制（也称 Replication， AB 复制）允许将来自一个MySQL数据库服务器（主服务器）的数据复制到一个或多个MySQL数据库服务器（从服务器）。

复制是异步的 从站不需要永久连接以接收来自主站的更新，从服务器甚至可以通过拨号断断续续地连接主服务器。

根据配置，可以复制数据库中的所有数据库，所选数据库甚至选定的表。

有以下几种形式：

* 一主一从
* 主主复制
* 一主多从---扩展系统读取的性能，因为读是在从库读取的；
* 多主一从---5.7开始支持
* 联级复制---

**MySQL中复制的优点：**

1，横向扩展解决方案 - 在多个从站之间分配负载以提高性能。在此环境中，所有写入和更新都必须在主服务器上进行。但是，读取可以在一个或多个从设备上进行。该模型可以提高写入性能（因为主设备专用于更新），同时显着提高了越来越多的从设备的读取速度。
2，数据安全性 - 因为数据被复制到从站，并且从站可以暂停复制过程，所以可以在从站上运行备份服务而不会破坏相应的主数据。
3，分析 - 可以在主服务器上创建实时数据，而信息分析可以在从服务器上进行，而不会影响主服务器的性能。
4，远程数据分发 - 您可以使用复制为远程站点创建数据的本地副本，而无需永久访问主服务器。

**主要过程**

1、主服务器MySQL服务将所有的写操作记录在 binlog 日志中，并生成 log dump 线程，将 binlog 日志传给从服务器MySQL服务的 I/O 线程。
2、从服务器MySQL服务生成两个线程，一个是 I/O 线程，另一个是 SQL 线程。
3、从库 I/O 线程去请求主库的 binlog 日志，并将 binlog 日志中的文件写入 relaylog（中继日志）中。
4、从库的 SQL 线程会读取 relaylog 中的内容，并解析成具体的操作，来实现主从的操作一致，达到最终两个数据库数据一致的目的。

![202051293523201.jpg](/img/202051293523201.jpg)

注意点：
- 主从复制是异步的逻辑的 SQL 语句级的复制；
- 复制时，主库有一个 I/O 线程，从库有两个线程，及 I/O 和 SQL 线程；
- 实现主从复制的必要条件是主库要开启记录 binlog 的功能；
- 作为复制的所有 MySQL 节点的 server-id 都不能相同；
- binlog 文件只记录对数据内容有更改的 SQL 语句，不记录任何查询语句。
- **无法将主服务器配置为仅记录特定事件。**

## 二进制日志

mysqld将数字扩展名附加到二进制日志基本名称以生成二进制日志文件名。每次服务器创建新日志文件时，该数字都会增加，从而创建一系列有序的文件。每次启动或刷新日志时，服务器都会在系列中创建一个新文件。服务器还会在当前日志大小达到 `max_binlog_size` 参数设置的大小后自动创建新的二进制日志文件 。二进制日志文件可能会比 `max_binlog_size` 使用大型事务时更大， 因为事务是以一个部分写入文件，而不是在文件之间分割。

为了跟踪已使用的二进制日志文件， mysqld还创建了一个二进制日志索引文件，其中包含所有使用的二进制日志文件的名称。默认情况下，它具有与二进制日志文件相同的基本名称，并带有扩展名 `.index`。在mysqld运行时，您不应手动编辑此文件。

`二进制日志文件` 通常表示包含数据库事件的单个编号文件。
`二进制日志` 表示含编号的二进制日志文件集加上索引文件。

SUPER 权限的用户可以使用 `SET sql_log_bin=0` 语句禁用其当前环境下自己的语句的二进制日志记录。

## 配置部署

**主从部署必要条件：**

* 主库开启binlog日志（设置log-bin参数）
* 主从server-id不同
* 从库服务器能连通主库

**主服务器：**

1、开启二进制日志
2、配置唯一的server-id
3、获得master二进制日志文件名及位置
4、创建一个用于slave和master通信的用户账号

**从服务器：**

1、配置唯一的server-id
2、使用master分配的用户账号读取master二进制日志
3、启用slave服务

### 服务器配置

**主修改配置文件**

```sh
# 服务器唯一ID，一般取IP最后一段
server-id=1

# 要同步的数据库(可以不指定)
binlog-do-db=test

# 要生成二进制日志文件 主服务器一定要开启
log-bin=/usr/local/mysql/binlogs/bin-log

# 不可以被从服务器复制的库
binlog-ignore-db=mysql
```

创建日志目录，并赋予权限：

```sh
mkdir /usr/local/mysql/binlogs
chown mysql.mysql /usr/local/mysql/binlogs
```

重启：`/etc/init.d/mysqld restart`

如果省略 `server-id`（或将其显式设置为默认值0），则主服务器拒绝来自从服务器的任何连接。

为了在使用带事务的 `InnoDB` 进行复制设置时尽可能提高持久性和一致性，
您应该在master `my.cnf` 文件中使用以下配置项：

```sh
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1
```

确保在主服务器上 `skip_networking` 选项处于 `OFF` 关闭状态, 这是默认值。
如果是启用的，则从站无法与主站通信，并且复制失败。
 
```sh
mysql> show variables like '%skip_networking%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| skip_networking | OFF   |
+-----------------+-------+
1 row in set (0.01 sec)
```

**从服务器配置**

```sh
# my.cnf 文件
[mysqld]
server-id=2

#要同步的数据库 (可选)
binlog-do-db=test

#要生成二进制日志文件（可选）
log-bin=mysql-bin
```

**创建用于复制数据的用户**

```sh
# 连接到mysql，输入密码
mysql -uroot -h 127.0.0.1 -p

# 创建用户
create user 'slave'@'%' identified by '123456';
# 或
CREATE USER 'slave'@'%'

# 设置用户权限
grant replication slave,replication client on *.* to 'slave'@'%';
# 或
GRANT REPLICATION SLAVE ON *.*  TO  'slave'@'%'  identified by '123456';

# 刷新权限
flush privileges;

# 查看用户权限
show grants for 'slave'@'%';
```

**备份主服务器数据**

```sh
# 如果不使用 --master-data 参数，则需要手动锁定单独会话中的所有表，如下所示。
mysqldump -u用户名 -p密码 --all-databases --master-data=1 > dbdump.db

# 1，加锁
FLUSH TABLES WITH READ LOCK;

# 2. 将master中需要同步的db的数据dump出来
mysqldump -uroot -p testdb > testdb.dump

# 3. 将数据导入slave
mysql -uroot -h192.123.75.69 -p testdb < testdb.dump

# 4. 解锁master
UNLOCK TABLES;
```

**从服务器开始复制**

```sh
# 登录 主mysql
mysql -uroot -h192.123.75.69 -p

# 主服务器复制状态
show master status;

# 登录从服务器
# 连接主服务器及设置复制的起始节点
change master to master_host='192.123.75.68',
master_port=3306,
master_user='slave',
master_password='123456',
master_log_file='bin-log.000001',
master_log_pos=154;

# 开始复制
start slave;

# 查看复制状态
show slave status \G

# 查看数据表数据
mysql> show create table user\G
```

复制相关命令：

```sh
# 停止slave
stop salve

# 重置slave
reset slave

# 开启slave
start slave

# 停止master
stop master

# 重置master
reset master

# 开启master
start master

# 查看主从服务器进程
show processlist;
```

参考：

[Mysql 主从复制](https://www.jianshu.com/p/faf0127f1cb2)

[MySQL 主从复制原理与实践详解](https://www.jb51.net/article/186349.htm)