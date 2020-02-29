---
title: SQL Server连接（内连接、外连接、交叉连接）
date: 2019-03-15 08:25:19
tags:
- 数据库
- Sql
- Sql Server
categories: 
- Sql
description: SQL Server连接（内连接、外连接、交叉连接）
---
# SQL Server连接（内连接、外连接、交叉连接）

连接查询概念：根据两个表或多个表的列之间的关系，从这些表中查询数据。

<font color=#ff0000 size=4 face="黑体">笛卡尔积：返回的是两表的乘积。</font>

首先，新建表：

```sql
--表结构
/*
Student(ID,Name,Age,Sex) 学生表
Course(ID,Name,TeacherID) 课程表
Score(StudentID,CourseID,Score) 成绩表
Teacher(ID,Name) 教师表
*/

CREATE TABLE [Student] (
  [ID] INTEGER NOT NULL PRIMARY KEY identity(1,1),
  [Name] NVARCHAR(20),
  [Age] INT, 
  [Sex] INT);

CREATE TABLE [Course] (
  [ID] INTEGER NOT NULL PRIMARY KEY identity(1,1), 
  [Name] NVARCHAR(20), 
  [TeacherID] INT);

CREATE TABLE [Score] (
  [Score] DOUBLE PRECISION,
  [StudentID] INT,
  [CourseID] INT);

CREATE TABLE [Teacher] (
  [ID] INTEGER NOT NULL PRIMARY KEY identity(1,1),  
  [Name] NVARCHAR(20))
```

## 内连接

注：INNER可省略

典型的联接运算，使用像 =  或 <> 之类的比较运算符。

显示的内连接，一般称为内连接，有INNER JOIN，形成的中间表为两个表经过ON条件过滤后的笛卡尔积。

示例：

```sql
 SELECT * FROM [Student] INNER JOIN Score ON Student.ID= Score.StudentID
```

![QQ截图20190315094830.png](/img/QQ截图20190315094830.png)

隐式的内连接，没有INNER JOIN，形成的中间表为两个表的笛卡尔积。

```sql
SELECT * FROM [Student],Score WHERE Student.ID= Score.StudentID
```

## 交叉连接

** 隐式交叉连接,没有CROSS JOIN**

```sql
SELECT * FROM [Student],Score WHERE Student.ID= Score.StudentID
```

** 显式交叉连接,使用CROSS JOIN**

```sql
SELECT * FROM [Student] CROSS JOIN Score WHERE Student.ID= Score.StudentID
```

以上两条sql 执行结果是相同的。

## 外连接

### 左连接

语法：`LEFT JOIN 或 LEFT OUTER JOIN`

```sql
SELECT * FROM [Student] LEFT OUTER JOIN Score ON Student.ID= Score.StudentID
```

<font color=#ff0000 size=4 face="黑体">左连接显示左表全部行,右表如果匹配，则显示;不匹配，显示Null。</font>

![QQ截图20190315095311.png](/img/QQ截图20190315095311.png)

### 右连接

语法：`RIGHT JOIN 或 RIGHT OUTER JOIN`

```sql
SELECT * FROM [Student] right OUTER JOIN Score ON Student.ID= Score.StudentID
```

<font color=#ff0000 size=4 face="黑体">右连接显示右表全部行,左表如果匹配，则显示;不匹配，显示Null。</font>

![QQ截图20190315095859.png](/img/QQ截图20190315095859.png)

### 全连接

语法：`FULL JOIN 或 FULL OUTER JOIN`

```sql
SELECT * FROM [Student]  full JOIN Score ON Student.ID= Score.StudentID
```

![QQ截图20190315100307.png](/img/QQ截图20190315100307.png)

<font color=#ff0000 size=4 face="黑体">返回左表和右表中的所有行</font>

参考：

[详解SQL Server连接（内连接、外连接、交叉连接）](https://blog.csdn.net/jiuqiyuliang/article/details/10474221)

[深入理解SQL的四种连接-左外连接、右外连接、内连接、全连接](https://www.jb51.net/article/39432.htm)