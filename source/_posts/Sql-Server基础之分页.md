---
title: Sql-Server基础之分页
date: 2019-03-14 10:41:55
tags:
- 数据库
- Sql
- Sql Server
categories: 
- Sql
description: Sql-Server分页方法总结
---
# Sql-Server基础之分页

测试sql语句执行时间：

```sql

declare @begin_date datetime
declare @end_date datetime
select @begin_date = getdate()

code

select @end_date = getdate()
select  datediff(ms,@begin_date,@end_date) as '毫秒'
```

## 定位法（ID大于多少）

```sql
SELECT TOP 10 * FROM dbo.[User] WHERE id >
(SELECT MAX(ID) FROM (SELECT TOP 1800000 id FROM dbo.[User] ORDER BY ID) AS t
) ORDER BY dbo.[User].ID
```

![QQ截图20190314112350.png](/img/QQ截图20190314112350.png)

先top 180W条，然后取10条。用时1020ms。用到了索引。

## 使用 Not In

```sql
select top 10 * from dbo.[User]
where id not in (select top 1800000 id from dbo.[User] order by id)
order by id
```

类似方法1，用时1170ms。用到了索引。

## Not Exists

```sql
select top 10 * from dbo.[User]
where not exists
(select 1 from (select top 1800000 id from dbo.[User] order by id) a  where a.id=dbo.[User].id)
order by id
```

注：`select 1` 表示记录；与 `select id`,`select *`相同，但占用的内存更小，效率较高。

类似not in ,用时1233ms;

## ROW_NUMBER()函数

** 用法1：**

```sql
select * from (
       select *,ROW_NUMBER() OVER (ORDER BY id) as rank from  dbo.[User]
)  as t where t.rank between 1800000 and 1800010
-- 或者 t.rank >1800000 and t.rank <=1800010
```

用时2106ms;

** 用法2：**

使用Top 优化，TOP函数让子查询每次仅返回必要的结果。

```sql
SELECT * FROM
(
    SELECT TOP (180001 * 10) ROW_NUMBER() OVER (ORDER BY id) AS RowNum, * 
    FROM dbo.[User] WHERE 1=1
) AS tempTable
WHERE RowNum > 1800000
ORDER BY RowNum
```

<font color=#ff0000 size=4 face="黑体">推荐</font>

## OFFSET FETCH子句分页

在SQL Server 2012 及以后的版本中终于出现了类似MySQL中LIMIT的写法了，那就是 OFFSET-FETCH 。

```sql
DECLARE @pageIndex INT
DECLARE @pageSize INT

SET @pageIndex=180001
SET @pageSize=10

SELECT * FROM dbo.[User] ORDER BY id 
OFFSET  ( @pageSize * ( @pageIndex - 1 )) ROWS 
FETCH NEXT @pageSize ROWS ONLY
```

参考：

[Sql Server 数据分页](http://www.cnblogs.com/qqlin/archive/2012/11/01/2745161.html)

[几种常见SQL分页方式效率比较](http://www.cnblogs.com/iamowen/archive/2011/11/03/2235068.html)