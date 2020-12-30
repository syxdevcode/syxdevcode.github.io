---
title: Mysql5.7.X启用安全审计功能
date: 2020-12-30 10:53:41
tags:
- Linux
- MySql
- CentOS7
- 安全审计
- 安装部署
categories: MySql
---

## 数据库审计简介

数据库审计（简称 DBAudit）能够实时记录网络上的数据库活动，对数据库操作进行细粒度审计的合规性管理，对数据库遭受到的风险行为进行告警，对攻击行为进行阻断。它通过对用户访问数据库行为的记录、分析和汇报，用来帮助用户事后生成合规报告、事故追根溯源，同时加强内外部数据库网络行为记录，提高数据资产安全。数据库审计可以记录某用户在某个时间点对数据库的操作，包括登录、连接、对表的增删改查等等，便于责任追溯，问题查找，当然开启审计功能在一定方面会影响数据库性能。

## 环境

* 操作系统：CentOS7.4
* Mysql：mysql-5.7.31-linux-glibc2.12-x86_64.tar.gz （社区版）
* Mariadb：mariadb-10.5.8-linux-x86_64

由于MySQL的社区版不支持审计系统，可通过第三方插件实现，此次采用MariaDB的server_audit插件来记录{时间，节点，用户，源IP，事件类型，库，语句，影响行数}，注：从mysql8开始已不支持该插件。

## 安装插件

1,登陆 Mysql，查看插件安装目录：

```sh
mysql> show global variables like 'plugin_dir';
+---------------+------------------------------+
| Variable_name | Value                        |
+---------------+------------------------------+
| plugin_dir    | /usr/local/mysql/lib/plugin/ |
+---------------+------------------------------+
1 row in set (0.01 sec)
```

2，提取 Mariadb 审计插件，并放置Mysql插件目录。

```sh
# 拷贝
cp /ldjc/mariadb-10.5.8-linux-x86_64/lib/plugin/server_audit.so /usr/local/mysql/lib/plugin/

# 授予执行权限
chmod  +x  /usr/local/mysql/lib/plugin/server_audit.so

# 安装
# 或者 在 my.cnf 中配置： server_audit=FORCE_PLUS_PERMANENT
install plugin server_audit soname 'server_audit.so';

# 备注：
# UNINSTALL PLUGIN server_audit; # 卸载插件
# show variables like '%audit%';

# 查看插件
show plugins;
```

3，新建 auditlogs 目录,并更改所属的组和用户

```sh
mkdir /usr/local/mysql/auditlogs
chown -R mysql /usr/local/mysql/auditlogs
chgrp -R mysql /usr/local/mysql/auditlogs
```

4，修改my.cnf配置文件

在 [mysqld] 标签下添加：

```sh
#防止server_audit 插件被卸载 进行配置文件配置
server_audit=FORCE_PLUS_PERMANENT

#指定哪些操作被记录到日志文件中
server_audit_events='CONNECT,QUERY,TABLE,QUERY_DDL'

#开启审计功能
server_audit_logging=on

#默认存放路径，可以不写，默认到data文件下
server_audit_file_path=/usr/local/mysql/auditlogs

#设置文件大小 默认1000000，1073741824=1GB
server_audit_file_rotate_size=1073741824

#指定日志文件的数量，如果为0日志将从不轮转
server_audit_file_rotations=200

#强制日志文件轮转
server_audit_file_rotate_now=ON
```

4，重启mysql

```sh
/etc/init.d/mysqld restart

# 或
service mysqld restart
```

## 验证

```sh
mysql> show variables like '%audit%';
+-------------------------------+-------------------------------+
| Variable_name                 | Value                         |
+-------------------------------+-------------------------------+
| server_audit_events           | CONNECT,QUERY,TABLE,QUERY_DDL |
| server_audit_excl_users       |                               |
| server_audit_file_path        | /usr/local/mysql/auditlogs    |
| server_audit_file_rotate_now  | ON                            |
| server_audit_file_rotate_size | 1073741824                    |
| server_audit_file_rotations   | 200                           |
| server_audit_incl_users       |                               |
| server_audit_loc_info         |                               |
| server_audit_logging          | ON                            |
| server_audit_mode             | 1                             |
| server_audit_output_type      | file                          |
| server_audit_query_log_limit  | 1024                          |
| server_audit_syslog_facility  | LOG_USER                      |
| server_audit_syslog_ident     | mysql-server_auditing         |
| server_audit_syslog_info      |                               |
| server_audit_syslog_priority  | LOG_INFO                      |
+-------------------------------+-------------------------------+
16 rows in set (0.00 sec)
```

```sh
cat /usr/local/mysql/auditlogs/server_audit.log
```

## 参数配置说明

* server_audit：启用安全审计插件；
* server_audit_output_type：指定日志输出类型，可为 SYSLOG或 FILE
* server_audit_logging：启动或关闭审计
* server_audit_events：指定记录事件的类型，可以用逗号分隔的多个值(connect,query,table)，如果开启了查询缓存(query cache)，查询直接从查询缓存返回数据，将没有 table 记录
* server_audit_file_path：如 server_audit_output_type 为 FILE，使用该变量设置存储日志的文件，可以指定目录，默认存放在数据目录的 server_audit.log 文件中
* server_audit_file_rotate_size：限制日志文件的大小
* server_audit_file_rotations：指定日志文件的数量，如果为0日志将从不轮转
* server_audit_file_rotate_now：强制日志文件轮转
* server_audit_incl_users：指定哪些用户的活动将记录，connect将不受此变量影响，该变量比server_audit_excl_users优先级高
* server_audit_syslog_facility：默认为LOG_USER，指定facility
* server_audit_syslog_ident：设置ident，作为每个syslog记录的一部分
* server_audit_syslog_info：指定的info字符串将添加到syslog记录
* server_audit_syslog_priority：定义记录日志的syslogd priority
* server_audit_excl_users：该列表的用户行为将不记录，connect将不受该设置影响
* server_audit_mode：标识版本，用于开发测试


参考：

[MariaDB Audit Plugin Options and System Variables](https://mariadb.com/kb/en/mariadb-audit-plugin-options-and-system-variables/)

[Mysql5.7安装server_audit审计](https://blog.csdn.net/endzhi/article/details/107317958)

[MySQL5.7审计功能windows系统](http://blog.itpub.net/31441024/viewspace-2213103)

[MySQL数据库启用安全审计功能](https://blog.csdn.net/weixin_39699061/article/details/103482490)


























