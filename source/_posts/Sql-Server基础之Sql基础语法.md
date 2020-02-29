---
title: Sql Server基础之Sql基础语法
date: 2019-03-14 08:45:54
tags:
- 数据库
- Sql
- Sql Server
categories: 
- Sql
---
# Sql Server基础之Sql基础语法

<font color=#ff0000 size=4 face="黑体">Sql： Structured Query Language</font>

** Sql server的组成**

* 主要数据库文件：.mdf 特点：有且只有一个
* 次要数据库文件：.ndf 特点：任意个
* 日志数据库文件：.ldf 特点：至少一个

** 操作数据库**

```sql
创建数据库：create databse 库名
查看所有数据库：exec sp_helpdb
查看当前数据库：exec sp_helpdb 库名
使用数据库：use 库名
删除数据库：drop database 库名 ps:正在使用的数据库无法删除
```

## 基础语法

### 创建表

```sql
CREATE TABLE [Student] (
  [ID] INTEGER NOT NULL PRIMARY KEY AUTOINCREMENT,
  [Name] NVARCHAR(20),
  [Age] INT,
  [Sex] INT);
```

### 查看所有表

`exec sp_help`

### 查看当前表

`exec sp_help 表名`

### 修改表结构

```sql
-- 1,增加一列语法
alter table 表名 add 字段名 数据类型

-- 2,修改一列语法
alter table 表名 alter column 字段名 数据类型

-- 3,删除一列语法
alter table 表名 drop column 字段名

-- 4,删除表语法
drop table 表名
```

### 操作表数据

```sql
-- 1,完全插入数据
insert into 表名 (字段名1,字段名2...) values(值1,值2...)

-- 2,省略插入数据
insert into 表名 values(值1,值2...)

-- 3，多行插入
insert into 表名 values(值1,值2...),(值1,值2...)

-- 4，查看所有记录
select * from 表名

-- 5，修改记录
update 表名 set 字段名=值 [where条件]

-- 6，删除一条记录
delete from 表名 where 条件

-- 7，清空表所有记录
delete from 表名
```

### 运算符

and , or ,not

## sql 查询

* 1，符合条件语法

    ```sql
    select 字段名 from 表名 [where 条件]
    ```

* 2，...到...之间

    ```sql
    and ... or
    between ... and
    --例句：查询23岁到25岁之间的学生
    select * from student where age>=23 and age<=25
    select * from student where age between 23 and 25
    ```

* 3，去重

    ```sql
    select distinct 字段名 from 表名
    ```

* 4,排序

    语法： order by + 字段名 asc升序（默认） desc降序

* 5，列别名 as
* 6，模糊查询 like
* 7, 联合查询 join

    ```sql
    -- 1,交叉查询
    select 字段名 from 表1 cross join 表2 [where 条件]

    -- 2,内连接查询
    select 字段名 from 表1 inner join 表2 on 联合条件 [where 条件]

    --外连接
    -- 3,左外连接
    select 字段名 from 表1 left join 表2 on 联合条件 [where 条件]

    -- 4，右外连接
    select 字段名 from 表1 right join 表2 on 联合条件 [where 条件]

    -- 5，全外连接
    select 字段名 from 表1 full join 表2 on 联合条件 [where 条件]
    ```

* 8,嵌套查询

    ```sql
    in() 在...范围之内的
    not in() 不在...范围之内的
    exists 存在
    not exists 不存在
    ```

* 9,分组 group by

## 系统函数

### 1，统计(聚合)函数

```sql
-- 1,AVG 返回指定组中的平均值，空值被忽略。
select prd_no,avg(qty) from sales group by prd_no

-- 2,COUNT 返回指定组中项目的数量
select count(prd_no) from sales

-- 3, MAX 返回指定数据的最大值。
select prd_no,max(qty) from sales group by prd_no

-- 4,MIN 返回指定数据的最小值。
select prd_no,min(qty) from sales group by prd_no

-- 5,SUM 返回指定数据的和，只能用于数字列，空值被忽略。
select prd_no,sum(qty) from sales group by prd_no

-- 6,COUNT_BIG 返回指定组中的项目数量，
-- 与COUNT函数不同的是COUNT_BIG返回bigint值，而COUNT返回的是int值。
select count_big(prd_no) from sales

-- 7,GROUPING 产生一个附加的列，当用CUBE或ROLLUP运算符添加行时，
-- 输出值为1.当所添加的行不是由CUBE或ROLLUP产生时，输出值为0.
select prd_no,sum(qty),grouping(prd_no) from sales group by prd_no with rollup

-- 8, BINARY_CHECKSUM 返回对表中的行或表达式列表计算的二进制校验值，用于检测表中行的更改。
select prd_no,binary_checksum(qty) from sales group by prd_no

-- 9,CHECKSUM_AGG 返回指定数据的校验值，空值被忽略。
select prd_no,checksum_agg(binary_checksum(*)) from sales group by prd_no

-- 10,CHECKSUM 返回在表的行上或在表达式列表上计算的校验值，用于生成哈希索引。

-- 11,STDEV 返回给定表达式中所有值的统计标准偏差。
select stdev(prd_no) from sales

-- 12,STDEVP 返回给定表达式中的所有值的填充统计标准偏差。
select stdevp(prd_no) from sales

-- 13, VAR 返回给定表达式中所有值的统计方差。
select var(prd_no) from sales

--14,VARP 返回给定表达式中所有值的填充的统计方差。
select varp(prd_no) from sales
```

### 2,日期函数

* 1， getDate()：获取当前时间
* 2，Dateadd()：增加时间

* 3，datediff(datepart,startdate,enddate)

    参考：[SQL Server DATEDIFF() 函数](http://www.w3school.com.cn/sql/func_datediff.asp)

    SELECT DATEDIFF(day,'2008-12-29','2008-12-30') AS DiffDate
    输出：1

    startdate 和 enddate 参数是合法的日期表达式。
    datepart 参数可以是下列的值：

    ```sql
    date part |缩写
    年:yy, yyyy
    季度:qq, q
    月:mm, m
    年中的日:dy, y
    日:dd, d
    周:wk, ww
    星期:dw, w
    小时:hh
    分钟:mi, n
    秒:ss, s
    毫秒:ms
    微妙:mcs
    纳秒：ns
    ```

### 3，数学函数

```sql
abs()：取绝对值
round()：四舍五入
floor()：函数返回小于或等于所给数字表达式的最大整数
ceiling()：函数返回大于或等于所给数字表达式的最小整数
sqrt()：开平方根
```

### 4，字符串函数

```sql
left()：左截串
right()：右截串
ltrim()：去左空格
rtrim()：去右空格
replace：(字符串,旧字符串,新字符串) 替换
substring：(字符串,位置,长度) 截字符串 ps:sql中字符串下标从1开始
reverse()：反转
len()：长度
upper()：转大写
lower()：转小写
```

## T-Sql

** 声明变量语法： **

```sql
declare @变量名 数据类型
set @变量名=值
select @变量名=值
```

** 编程语句:**

```sql
begin...end
if...else
```

## 视图

* 1.创建视图

    ```sql
    create view 视图名称
    as
    sql中查询语句
    ```

* 2.使用视图 select * from 视图名

* 3.查看视图 exec sp_help

* 4.查看视图内容 exec sp_helptext 视图名

* 5.修改视图 alter view 视图名 as select * from 表名 [where条件]

* 6.删除视图 drop view 视图名

* 7.修改视图 update 视图名 set 字段名=值 [where条件]

## 存储过程

```sql
create proc | procedure pro_name
    [{@参数数据类型} [=默认值] [output],
     {@参数数据类型} [=默认值] [output],
     ....
    ]
as
        select .....
```

参考：

[sql server 基础教程[温故而知新三]](http://www.cnblogs.com/toutou/p/4733670.html)

[.NET面试题解析(11)-SQL语言基础及数据库基本原理](http://www.cnblogs.com/anding/p/5281558.html)