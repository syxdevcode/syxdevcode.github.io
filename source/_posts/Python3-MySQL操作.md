---
title: Python3-MySQL操作
date: 2021-04-28 17:16:51
tags:
- Python3
- Python
categories: Python3
---


5 种方式操作 MySQL。

**MySQL-python**
MySQL-python 也称 MySQLdb，基于 C 库开发，曾经是一个十分流行的 MySQL 驱动，具有出色的性能，但其早已停更，仅支持 Python2，不支持 Python3，现在基本不推荐使用了，取而代之的是它的衍生版。

**mysqlclient**
MySQLdb 的 Fork 版本，完全兼容 MySQLdb，支持 Python3，它是 Django ORM 的依赖工具，如果你喜欢用原生 SQL 操作数据库，那么推荐使用它。

**PyMySQL**
PyMySQL 采用纯 Python 开发，兼容 MySQLdb，性能不如 MySQLdb，安装方便，支持 Python3。

**peewee**
peewee 是一个流行的 ORM 框架，实现了对象与数据库表的映射，兼容多种数据库，我们无需知道原生 SQL，只要了解面向对象的思想就可以简单、快速的操作相应数据库，支持 Python3。

**SQLAlchemy**
SQLAlchemy 是一个 ORM 框架，同时也支持原生 SQL，支持 Python3，它类似于 Java 的 Hibernate 框架。

<!--more-->

## mysqlclient

安装mysqlclient `pip install mysqlclient`

```py
import MySQLdb

connect = MySQLdb.connect(
    # 主机
    host='192.168.120.77',
    # 端口号
    port=3306,
    # 用户名
    user='root',
    # 密码
    passwd='test1',
    # 数据库名称
    db='platform_business',
    # 指定字符的编、解码格式
    use_unicode=True,
    charset='utf8')
# 先获取游标，再进行相应 SQL 操作
cursor = connect.cursor()

# 增删改查操作

# 关闭
cursor.close()
connect.close()
```

**新增**

```py
# 执行新增 SQL
sql = 'insert into c_index (name, code,type) values(%s,%s,%s);'
data = [
    ('学历', 'AAC011',1),
    ('经营方式', 'AAB048',1)
]
cursor.executemany(sql, data)
# 提交
connect.commit()
```

**查询**

cursor 查看方法

* fetchone() 获取结果集的下一行
* fetchmany(size) 获取结果集的下几行务
* fetchall() 获取结果集中剩下的所有行

```py
cursor.execute('SELECT * FROM c_index')
print(cursor.fetchall())
```

**修改**

```py
cursor.execute("UPDATE c_index SET name='学历' WHERE id = 1")
connect.commit()
```

**删除**

```py
cursor.execute("DELETE FROM c_index WHERE id = 1")
connect.commit()
```

## PyMySQL

安装 pymysql `pip install pymysql`，使用方式与 mysqlclient 基本类似。

```py
import pymysql

connect = pymysql.connect(
    host='192.168.120.77',
    port=3306,
    user='root',
    password='test1',
    database='platform_business',
    charset='utf8'
)
cursor = connect.cursor()

# 执行新增 SQL
sql = 'insert into c_index (name, code,type) values(%s,%s,%s);'
data = [
    ('学历', 'AAC011', 1),
    ('经营方式', 'AAB048', 1)
]
cursor.executemany(sql, data)
# 提交
connect.commit()

# 查询
cursor.execute('SELECT * FROM c_index')
print(cursor.fetchall())

# 关闭
cursor.close()
connect.close()
```

## peewee

安装 `pip install peewee`

**定义映射类**

```py
from peewee import *

# 连接数据库
db = MySQLDatabase('test',
                   host='localhost',
                   port=3306,
                   user='root',
                   passwd='root',
                   charset='utf8')

# 映射类
class Teacher(Model):
    id = AutoField(primary_key=True)
    name = CharField()
    age = IntegerField()

    class Meta:
        database = db

## 增删改查操作
```

**创建表**

```py
db.create_tables([Teacher])
```

**新增**

```py
t1 = Teacher(name='张三', age=22)
t2 = Teacher(name='李四', age=33)
t3 = Teacher(name='王五', age=33)
t1.save()
t2.save()
t3.save()
```

**查询**

```py
# 查询单条数据
t = Teacher.get(Teacher.id == 1)
print('name：', t.name)
# 查询多条
ts = Teacher.select().where(Teacher.age == 33)
for t in ts:
    print('name：', t.name)
```

**修改**

```py
t = Teacher.update({Teacher.name: '张四'}).where(Teacher.id == 1)
t.execute()
```

**删除**

```py
Teacher.delete().where(Teacher.id == 2).execute()
```

## SQLAlchemy

安装 SQLAlchemy `pip install sqlalchemy`

**定义映射类**

```py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String

engine = create_engine('mysql+mysqldb://root:root@localhost:3306/test?charset=utf8',
                       # 打印执行语句
                       echo=True,
                       # 连接池大小
                       pool_size=10,
                       # 指定时间内回收连接
                       pool_recycle=3600)
# 映射基类
Base = declarative_base()
# 具体映射类

class Teacher(Base):
    # 指定映射表名
    __tablename__ = 'teacher'
    # 映射字段
    id = Column(Integer, primary_key=True)
    name = Column(String(30))
    age = Column(Integer)

## 增删改查操作
```

**创建表**

```py
Base.metadata.create_all(engine)
```

**新增**

```py
Session = sessionmaker(bind=engine)
# 创建 Session 类实例
session = Session()
ls = []
t1 = Teacher(name='张三', age=22)
t2 = Teacher(name='李四', age=33)
t3 = Teacher(name='王五', age=33)
ls.append(t1)
ls.append(t2)
ls.append(t3)
session.add_all(ls)
session.commit()
session.close()
```

**查询**

```py
Session = sessionmaker(bind=engine)
# 创建 Session 类实例
session = Session()
# 查询一条数据，filter 相当于 where 条件
t = session.query(Teacher).filter(Teacher.id==1).one()
print('---', t.name)
# 查询所有数据
ts = session.query(Teacher).filter(Teacher.age==33).all()
for t in ts:
    print('===', t.name)
```

**修改**

```py
Session = sessionmaker(bind=engine)
# 创建 Session 类实例
session = Session()
t = session.query(Teacher).filter(Teacher.id==1).one()
print('修改前名字-->', t.name)
t.name = '张四'
session.commit()
print('修改后名字-->', t.name)
```

**删除**

```py
Session = sessionmaker(bind=engine)
# 创建 Session 类实例
session = Session()
t = session.query(Teacher).filter(Teacher.id==1).one()
session.delete(t)
session.commit()
```

参考：

[Python 进阶（四）：数据库操作之 MySQL](https://ityard.blog.csdn.net/article/details/104323444)

