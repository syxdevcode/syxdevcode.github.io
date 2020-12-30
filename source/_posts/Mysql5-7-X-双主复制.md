---
title: Mysql5.7.X 双主复制
date: 2020-12-29 16:57:53
tags:
- Linux
- MySql
- CentOS7
- 安装部署
categories: MySql
---

## 修改配置文件

前提：

* 1，两台服务器都需要添加一个用户数据同步的帐号。 
* 2，端口，策略，数据库远程等互通。
* 3，主库备份后，从库需要还原备份文件。

配置以下两台服务器互为主从。

配置：`vim /usr/local/mysql/my.cnf`

主1服务器配置：

```sh
auto_increment_increment=2         #步进值 auto_imcrement。一般有n台主MySQL就填n
auto_increment_offset=1            #起始值。一般填第n台主MySQL。此时为第一台主MySQL
binlog-ignore-db=mysql                #忽略mysql库【可忽略不写】
binlog-ignore-db=information_schema   #忽略information_schema库【可忽略不写】
binlog-ignore-db=performance_schema   #忽略performance_schema库【可忽略不写】
log-slave-updates=on
```

主2服务器配置：

```sh
auto_increment_increment=2         #步进值auto_imcrement。一般有n台主MySQL就填n
auto_increment_offset=2            #起始值。一般填第n台主MySQL。此时为第二台主MySQL
binlog-ignore-db=mysql                #忽略mysql库【可忽略不写】
binlog-ignore-db=information_schema   #忽略information_schema库【可忽略不写】
binlog-ignore-db=performance_schema   #忽略performance_schema库【可忽略不写】
log-slave-updates=on
```

主1完整配置：

```sh
[mysql]
socket=/var/lib/mysql/mysql.sock
# set mysql client default chararter
default-character-set=utf8
#
[mysqld]
socket=/var/lib/mysql/mysql.sock
bind-address=0.0.0.0
# set mysql server port  
port = 3306 #默认是3306，防止这种情况发生，可以避免使用3306mysql默认端口
# set mysql install base dir
basedir=/usr/local/mysql
# set the data store dir
datadir=/usr/local/mysql/data
# set the number of allow max connnection
max_connections=200000
# set server charactre default encoding
character-set-server=utf8
# the storage engine
default-storage-engine=INNODB
# lower_case_table_names=1
max_allowed_packet=16M
explicit_defaults_for_timestamp=true

sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

# 服务器唯一ID
server-id=1

# 要生成二进制日志文件 主服务器一定要开启
log-bin=/usr/local/mysql/binlogs/bin-log

binlog_format=ROW
expire_logs_days=7
max_binlog_size=100M
binlog_cache_size=4M
max_binlog_cache_size=512M

auto_increment_increment=2         
auto_increment_offset=1            
binlog-ignore-db=mysql             
binlog-ignore-db=information_schema
binlog-ignore-db=performance_schema

log-slave-updates=on

#skip-grant-tables=1

[mysql.server]
user=mysql
basedir=/usr/local/mysql
```

## 配置主从

互相配置主从服务器。

主1服务器：192.123.75.68；
主2服务器：192.123.3.87；

登陆到主1服务器，执行一下命令：

```sh
# 查看 主 复制状态
show master status \G

change master to master_host='192.123.3.87',
master_port=3306,
master_user='slave',
master_password='123456',
master_log_file='bin-log.000005',
master_log_pos=74562307;

# 开始复制
start slave;

# 查看 从 复制状态
# 输出结果中应该看到 I/O 线程和 SQL 线程都是 YES, 就表示成功。
show slave status \G

# 主服务器复制状态
show master status\G
```

登陆到主2服务器，执行一下命令：

```sh
# 查看 主 复制状态
show master status \G

change master to master_host='192.123.75.68',
master_port=3306,
master_user='slave',
master_password='123456',
master_log_file='bin-log.000005',
master_log_pos=74562307;

# 开始复制
start slave;

# 查看 从 复制状态
# 输出结果中应该看到 I/O 线程和 SQL 线程都是 YES, 就表示成功。
show slave status \G

# 主服务器复制状态
show master status\G
```
