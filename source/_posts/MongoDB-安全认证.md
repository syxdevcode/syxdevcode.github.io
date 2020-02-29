---
title: MongoDB 安全认证
date: 2018-08-24 11:37:04
tags:
- Linux
- MongoDB
- CentOS7
categories: 
- MongoDB
---
# mongodb 安全认证

## MongoDB权限介绍

1.MongoDB安装时不添加任何参数,默认是没有权限验证的,登录的用户可以对数据库任意操作而且可以远程访问数据库，需以--auth参数启动。

2.在刚安装完毕的时候MongoDB都默认有一个admin数据库,此时admin数据库是空的,没有记录权限相关的信息。当admin.system.users一个用户都没有时，即使mongod启动时添加了--auth参数,如果没有在admin数据库中添加用户,此时不进行任何认证还是可以做任何操作(不管是否是以--auth 参数启动),直到在admin.system.users中添加了一个用户。

3.MongoDB的访问分为连接和权限验证，即使以--auth参数启动还是可以不使用用户名连接数据库，但是不会有任何的权限进行任何操作

4.admin数据库中的用户名可以管理所有数据库，其他数据库中的用户只能管理其所在的数据库。

## MongoDB中用户的角色说明

1,read角色

数据库的只读权限

2,readWrite角色

数据库的读写权限

3,dbAdmin角色

数据库的管理权限

不具备读写权限

4,userAdmin角色

数据库的用户管理权限

5,clusterAdmin角色

集群管理权限(副本集、分片、主从等相关管理)

6,readAnyDatabase角色

任何数据库的只读权限(和read相似)

7,readWriteAnyDatabase角色

任何数据库的读写权限(和readWrite相似)

8,userAdminAnyDatabase角色

任何数据库用户的管理权限(和userAdmin相似)

9,dbAdminAnyDatabase角色

任何数据库的管理权限(dbAdmin相似)

## 启用访问控制和强制认证

### 在admin数据库中，创建一个admin 用户

```bash
use admin
db.createUser({
    user:"admin",
    pwd:"admin",
    roles:[{
        role:"root",
        db:"admin"
    }]
})
db.auth("admin", "admin")
```

### 对具体的库进行授权

```bash
use mydb
db.createUser({
    user:"mydbuser1",
    pwd:"123456",
    roles:[{
        role:"dbAdmin",
        db:"mydb"
    },{
        role:"readWrite",
        db:"mydb"
    }]
})
db.auth("mydbuser1", "123456")
```

## 常用命令

```bash
// 添加用户
use admin
db.addUser("test","test")

// 显示所有数据库
show dbs

// 使用某个数据库
use mydb

// 连接数据库
mongo test -uroot -p123456

// 添加用户认证
db.auth("username","password")

// 查看用户
db.system.users.find()
```

参考：

[MongoDB安全配置](https://wps2015.org/drops/drops/MongoDB%E5%AE%89%E5%85%A8%E9%85%8D%E7%BD%AE.html)

[MongoDB 3.4.2 添加用户、设置权限](https://blog.csdn.net/zZ_life/article/details/78664794)

[https://docs.mongodb.com/manual/tutorial/enable-authentication/](https://docs.mongodb.com/manual/tutorial/enable-authentication/)