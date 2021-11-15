---
title: Sql-Server基础之常用sql实例
date: 2019-03-14 16:03:28
tags:
- 数据库
- Sql
- Sql Server
categories: 
- Sql
description: Sql-Server常用sql实例总结
---

表结构：

```sql
Student(ID,Name,Age,Sex) 学生表
Course(ID,Name,TeacherID) 课程表
Score(StudentID,CourseID,Score) 成绩表
Teacher(ID,Name) 教师表
```

## 实例

**1，查询语文“1”比数学“2”课程成绩高的所有学生的姓名**

先查询语文，数学的单科成绩，然后根据过滤条件，连表查询。

```sql
select st.[Name],c1.Score,c2.Score from
(select sc.[Score], sc.StudentID from Score sc  where sc.[CourseID]=1)c1,
(select sc.[Score],sc.StudentID from Score sc where sc.[CourseID]=2)c2
join Student st on st.[ID]= c2.[StudentID]
where c1.[Score]>c2.[Score] and c1.[StudentID]=c2.[StudentID]

-- 与上面sql执行结果相同
select stu.[Name],c1.Score,c2.Score from
(select sc.[Score], sc.StudentID from Score sc  where sc.[CourseID]=1)c1,
(select sc.[Score],sc.StudentID from Score sc where sc.[CourseID]=2)c2,student stu
WHERE c1.[StudentID]=c2.[StudentID] AND c1.[Score]>c2.[Score] AND stu.ID=c1.StudentID
```

**2. 查询平均成绩大于60分的同学的学号和平均成绩**

在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。
<font color=#ff0000 size=4 face="黑体">GROUP BY后面不能接WHERE条件，使用HAVING代替</font>

先以学生ID分组，然后求每个学生的平均成绩。

```sql
SELECT StudentID,AVG(stu.score) FROM  [dbo].[Score] AS stu 
GROUP BY StudentID HAVING AVG(stu.score)>60
```

**3. 查询所有同学的学号、姓名、选课数、总成绩；**

成绩表分组统计总成绩，课程数目，然后连表查姓名。

```sql
SELECT sc1.StudentID,sc.NAME,counts,sums FROM Student sc JOIN 
(
SELECT StudentID AS StudentID,COUNT(CourseID) AS counts,SUM(score) sums 
   FROM Score stu GROUP BY StudentID
) as sc1 ON sc1.StudentID= sc.ID

--不显示姓名可以用一下sql
SELECT stu.ID ,COUNT(sc.CourseID),SUM(score) FROM Student stu 
LEFT OUTER JOIN Score sc ON  sc.StudentID= stu.ID
GROUP BY stu.ID
```

外连接的三种形式如下表，其中outer可以省略。与外连接对应的就是内连接inner join ，要两个表同时满足指定条件。

![151257-20160315222925412-1484058927.png](/img/151257-20160315222925412-1484058927.png)

**4. 查询姓“张”的老师的个数；**

```sql
select count(t.ID) from Teacher t where t.Name like '张%'
```

SQL LIKE子句使用通配符运算符比较相似的值。符合LIKE操作符配合使用2个通配符：

百分号 (%)：百分号代表零个，一个或多个字符

下划线 (_)：下划线表示单个数字或字符

**5. 找出教师表中姓名重复的数据，然后删除多余重复的记录，只留ID小的那个。**

```sql
-- 首先找到重复姓名的
select t.Name,count(t.Name) from Teacher t group by t.[Name] having count(t.Name)>1

delete from Teacher where Name in
       (select t2.Name from Teacher t2 group by t2.[Name] having count(t2.Name)>1)
and ID not in
       (select min(t3.ID) from Teacher t3 group by t3.Name having count(t3.Name)>1)
```

**6. 按照成绩分段标示（<60不及格，60-80良，>80优），输出所有学生姓名、课程名、成绩、成绩分段标示。**

```sql
select st.Name,c.Name,sc.Score,(case
       when sc.Score > 80 then '优'
       when sc.Score < 60 then '不及格'
       else '良' end) as 'Remark'
from Score sc
inner join Student st on st.ID=sc.[StudentID]
inner join Course c on c.ID=sc.[CourseID]
ORDER BY st.Name
```

**7. 查询所有课程成绩小于60分的同学的学号、姓名信息**

```sql
-- 先查询小于60分的学生，然后去重
select distinct st.* from Student st
left join Score sc on sc.[StudentID]=st.ID
where sc.[Score]<60

-- 查询小于60分的学生作为子查询；
select * FROM Student st WHERE st.ID in 
       (SELECT s1.ID FROM Student s1,Score s2 WHERE s1.ID=s2.[StudentID] AND s2.Score<60)
```

**8. 查询各科成绩最高和最低的分：以如下形式显示：课程名称，最高分，最低分**

```sql
SELECT cou.NAME AS 课程名称,maxsc 最高分,minsc 最低分  FROM
[dbo].[Course] cou JOIN
(
SELECT [CourseID],MAX(sc.score) maxsc ,MIN(sc.score) minsc
 FROM Score sc,[dbo].[Student] stu group BY [CourseID]
) tb1 ON cou.id=tb1.[CourseID]
```

**9. 查询不同老师所教不同课程平均分从高到低显示**

```sql
SELECT te.NAME,cu.NAME,cu.TeacherID,sc1.avgsc FROM [dbo].[Course] cu JOIN 
(
SELECT sc.[CourseID] ,AVG(sc.Score) avgsc FROM Score sc,[dbo].[Teacher] te
 GROUP BY sc.[CourseID]
) sc1  ON sc1.[CourseID]= cu.ID
JOIN [dbo].Teacher te ON te.id= cu.[TeacherID]
ORDER BY avgsc desc
```

![QQ截图20190315144406.png](/img/QQ截图20190315144406.png)

**10. 查询和“7”号的同学学习的课程完全相同的其他同学学号和姓名**

```sql
SELECT stu.id,stu.Name  FROM Student stu JOIN 
(
select s1.StudentID from Score s1 where s1.CourseID IN
(select s2.CourseID from Score s2 where s2.StudentID=7)
group by s1.StudentID 
having count(*)=(select count(*) from Score s3 where s3.StudentID=7)
) sc ON stu.id=sc.studentid
```

**11. 查询选修“张老师”老师所授课程的学生中，成绩最高的学生姓名及其成绩**

```sql
SELECT cou.NAME,te.NAME,sc.score,sc.StudentID,stu.name FROM
[dbo].[Course] cou,[dbo].[Teacher] te,[dbo].[Score] sc,student stu
where  te.name= '张老师' AND cou.TeacherID=te.ID AND cou.id=sc.courseid
AND sc.score= (SELECT MAX(score) FROM [dbo].[Score] sc1 WHERE sc1.[CourseID]=cou.ID)
AND stu.id =sc.StudentID
```

![QQ截图20190316134906.png](/img/QQ截图20190316134906.png)

**12. 查询所有成绩第二名到第四名的成绩**

```sql
-- sqlite 分页
select  s.[StudentID],s.Score from Score s order by s.Score desc limit 2 offset 2
-- SQL 2005/2008中的分页函数是ROW_NUMBER() Over (Order by 列...)--
select  t.[StudentID],t.Score from(
        select s2.[StudentID],s2.Score,ROW_NUMBER() OVER (ORDER BY s2.[Score]) AS rn from Score s2) t
where t.rn>=2 and t.rn<=4
```

**12. 查询各科成绩前2名的记录:(不考虑成绩并列情况)**

```sql
select * from Score s1 where s1.Score 
       in(select s2.Score from Score s2 where s1.[CourseID]=s2.[CourseID] order by s2.Score desc limit 2 offset 0)
order by s1.[CourseID],s1.[Score] desc
-- 上面是sqlite中的语法，sqlite中没有top，使用limit代替，效果是一样的 --
select * from Score s1 where s1.Score 
       in(select Top 2 s2.Score from Score s2 where s1.[CourseID]=s2.[CourseID] order by s2.Score desc)
order by s1.[CourseID],s1.[Score] desc
```

**表A，表B，使用表B中的Name字段更新表A中的Name字段，其中Id字段相关联**

```sql
update   b  set   ClientName   =   a.name   from   a,b   where   a.id   =   b.id

--oracle
update   b  set   (ClientName)   =  (SELECT name FROM a WHERE b.id = a.id)

--mysql
UPDATE A, B SET A1 = B1, A2 = B2, A3 = B3 WHERE A.ID = B.ID
```

参考：

[.NET面试题解析(11)-SQL语言基础及数据库基本原理](http://www.cnblogs.com/anding/p/5281558.html)

[SQL常考笔试题[转]](http://www.cnblogs.com/2719-feng/archive/2011/08/31/2160434.html)