---
title: Python3-操作SqLite
date: 2021-04-28 21:09:14
tags:
- Python3
- Python
categories: Python3
---

## 基本使用

```py
import sqlite3

# 连接数据库
conn = sqlite3.connect('test.db')

# 创建游标
cs = conn.cursor()

## 增删改查操作

# 关闭 cursor
cs.close()

# 关闭连接
conn.close()
```

<!--more-->
**创建表**

```py
sql = "CREATE TABLE person(id varchar(20) PRIMARY KEY, name varchar(20));"

# 创建表
cs.execute(sql)

# 提交当前事务
conn.commit()
```

```py
# 新增
cs.execute("INSERT INTO person (id, name) VALUES ('1', '张三')")
cs.execute("INSERT INTO person (id, name) VALUES ('2', '李四')")
cs.execute("INSERT INTO person (id, name) VALUES ('3', '王五')")
cs.execute("INSERT INTO person (id, name) VALUES ('4', '赵六')")
cs.execute("INSERT INTO person (id, name) VALUES ('5', '朱七')")

# 提交当前事务
conn.commit()
```

**查询**

```py
cs.execute("SELECT id, name FROM person")

# 获取查询结果集中的下一行
print(cs.fetchone())

# 获取查询结果集中的下几行
print(cs.fetchmany(2))

# 获取查询结果集中剩下的所有行
print(cs.fetchall())
```

**修改**

```py
# 修改
cs.execute("UPDATE person set name = '张四' WHERE id = '1'")
conn.commit()
```

**删除**

```py
# 删除
cs.execute("DELETE FROM person WHERE id = '3'")
conn.commit()
```

参考：

[Python 进阶（五）：数据库操作之 SQLite](https://ityard.blog.csdn.net/article/details/104338259)