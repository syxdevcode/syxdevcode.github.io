---
title: mysql启动报错总结
date: 2020-07-31 09:32:40
tags:
- Linux
- MySql
- CentOS7
- 安装部署
categories: MySql
---

## [ERROR] InnoDB: redo log file './ib_logfile0' exists

```log
2020-07-31T01:24:43.755177Z 0 [ERROR] InnoDB: redo log file './ib_logfile0' exists. Creating system tablespace with existing redo log files is not recommended. Please delete all redo log files before creating new system tablespace.
2020-07-31T01:24:43.755184Z 0 [ERROR] InnoDB: InnoDB Database creation was aborted with error Generic error. You may need to delete the ibdata1 file before trying to start up again.
2020-07-31T01:24:44.355412Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2020-07-31T01:24:44.355420Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2020-07-31T01:24:44.355425Z 0 [ERROR] Failed to initialize builtin plugins.
2020-07-31T01:24:44.355429Z 0 [ERROR] Aborting
```

问题解决如下

**1.如果数据不重要或已经有备份，只需要恢复mysql启动 (已验证可用)**

进入mysql目录，一般是：/usr/local/mysql/data
删除ib_logfile*
删除ibdata*
删除所有数据库物理目录（例如数据库为test_db,则执行rm -rf test_db）
重启动mysql
重新建立数据库或使用备份覆盖

**2.如果数据很重要且没有备份**

可以使用innodb_force_recovery参数，使mysqld跳过恢复步骤，启动mysqld，将数据导出然后重建数据库。

innodb_force_recovery 可以设置为1-6，大的数字包含前面所有数字的影响

(SRV_FORCE_IGNORE_CORRUPT):忽略检查到的corrupt页。

(SRV_FORCE_NO_BACKGROUND):阻止主线程的运行，如主线程需要执行full purge操作，会导致crash。

(SRV_FORCE_NO_TRX_UNDO):不执行事务回滚操作。

(SRV_FORCE_NO_IBUF_MERGE):不执行插入缓冲的合并操作。

(SRV_FORCE_NO_UNDO_LOG_SCAN):不查看重做日志，InnoDB存储引擎会将未提交的事务视为已提交。

(SRV_FORCE_NO_LOG_REDO):不执行前滚的操作。

在my.cnf(windows是my.ini)中加入 
innodb_force_recovery = 6 
innodb_purge_thread = 0（purge多线程参数）

重启mysql

这时只可以执行 `select,create,drop` 操作，但不能执行 `insert,update,delete` 操作 
执行逻辑导出，导出需要的数据库

完成后将innodb_force_recovery=0，innodb_purge_threads=1，

然后重建数据库，最后把导出的数据重新导入。

参考：

[[ERROR] InnoDB: Plugin initialization aborted with error Generic error几种报错](https://blog.csdn.net/yunhua12/article/details/81393572)

[启动mysql报错The server quit without updating PID file！](https://cloud.tencent.com/developer/article/1409737)