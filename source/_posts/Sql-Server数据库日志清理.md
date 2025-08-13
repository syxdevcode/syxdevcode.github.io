---
title: Sql-Server数据库日志清理
date: 2022-10-12 14:19:19
tags:
  - Sql Server
  - 数据库
  - Sql
categories:
  - Sql Server
---

查看Log文件名：选择数据库，属性，查看文件。

![Snipaste_2022-10-12_14-23-46.png](/img1/Snipaste_2022-10-12_14-23-46.png)

```sql
USE [master]
GO
ALTER DATABASE 要清理的数据库名称 SET RECOVERY SIMPLE WITH NO_WAIT
GO
ALTER DATABASE 要清理的数据库名称 SET RECOVERY SIMPLE   --简单模式
GO
USE 要清理的数据库名称
GO

-- 注意log名称，需要使用文件逻辑名称
DBCC SHRINKFILE (N'要清理的数据库名称_log' , 2, TRUNCATEONLY)  --设置压缩后的日志大小为2M，可以自行指定
GO
USE [master]
GO
ALTER DATABASE 要清理的数据库名称 SET RECOVERY FULL WITH NO_WAIT
GO
ALTER DATABASE 要清理的数据库名称 SET RECOVERY FULL  --还原为完全模式
GO
```