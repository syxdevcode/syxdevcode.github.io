---
title: Sql-Server基础之Grouping用法
date: 2019-03-16 14:51:00
tags:
- 数据库
- Sql
- Sql Server
categories: 
- Sql
---
# Sql-Server基础之Grouping用法

`GROUPING` 是一个聚合函数，用在含有 `CUBE` 或 `ROLLUP` 语句的SQL语句中，当结果集中的数据行是由 `CUBE` 或 `ROLLUP` 运算产生的（添加的）则该函数返回1，否则返回0。

语法： `GROUPING(column_name)`

其中 `column_name` 是用在 `CUBE` 或 `ROLLUP` 运算的列 或 `group by` 后的列。

注意：

* （1）只有使用了 `CUBE` 或 `ROLLUP` 运算符的SQL中才能使用 `GROUPING`
* （2）`GROUPING` 后面的列 名可以是 `CUBE` 或 `ROLLUP` 运算符中使用的列名，也可以是 `group by` 中的列名

CUBE 生成的结果集显示了所选列中值的所有组合的聚合。
ROLLUP 生成的结果集显示了所选列中值的某一层次结构的聚合。

** CUBE**

`CUBE` 生成的结果集显示了所选列中值的所有组合的聚合。

```sql
SELECT CourseID,StudentID,SUM(Score) FROM [dbo].[Score] 
GROUP BY CourseID,StudentID WITH CUBE
```

结果：
![QQ截图20190316155035.png](/img/QQ截图20190316155035.png)

** ROLLUP**

```sql
SELECT StudentID,CourseID,SUM(Score)  FROM [dbo].[Score] 
GROUP BY StudentID,CourseID WITH ROLLUP
```

结果：
![QQ截图20190316160822.png](/img/QQ截图20190316160822.png)

** GROUPING**

获取每个学生成绩的总和。以下示例结果相同

```sql
SELECT StudentID,CourseID,SUM(Score),grouping(CourseID) FROM [dbo].[Score] 
GROUP BY StudentID,CourseID WITH CUBE HAVING grouping(CourseID)=1
```

结果：
![QQ截图20190316161849.png](/img/QQ截图20190316161849.png)

```sql
SELECT StudentID,CourseID,SUM(Score),grouping(CourseID)  FROM [dbo].[Score] 
GROUP BY StudentID,CourseID WITH ROLLUP HAVING grouping(CourseID)=1
```

结果：
![QQ截图20190316162226.png](/img/QQ截图20190316162226.png)

参考：

[Sql学习第四天——SQL 关于with cube ，with rollup 和 grouping](https://www.cnblogs.com/shuangnet/archive/2013/03/26/2982144.html)

[GROUPING 用法](https://www.cnblogs.com/dyufei/archive/2009/11/12/2573973.html)