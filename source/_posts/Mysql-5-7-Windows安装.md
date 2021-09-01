---
title: Mysql-5.7-Windows安装
date: 2021-08-31 09:40:06
tags:
- Windows
- MySql
- Windows10
- 安装部署
categories: MySql
---

## 添加环境变量

我的电脑->属性->高级->环境变量
选择PATH,在其后面添加: mysql 安装文件下面的bin文件夹
(如: `D:\mysql-5.7.34-winx64\bin` )(注意不要删除其他的东西)

4.新建 my.ini 文件
5.编辑my.ini文件

```ini
[mysqld]
basedir=D:\mysql-5.7.34-winx64\
datadir=D:\mysql-5.7.34-winx64\data\
port=3306
skip-grant-tables

#basedir表示mysql安装路径
#datadir表示mysql数据文件存储路径
#port表示mysql端口
#skip-grant-tables表示忽略密码
```

6.启动管理员模式下的 CMD，并将路径切换至mysql下的bin目录，然后输入

```sh
mysqld –install
```

7.再输入 `mysqld --initialize-insecure;` 初始化数据文件

8.输入 `net start mysql` 启动mysql服务

注意：data文件必须存在

9.然后再次启动 mysql 然后用命令 `mysql –u root –p` 进入mysql管理界面（密码可为空）

10.进入界面后更改root密码

```sh
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root';

# 或者
update mysql.user set authentication_string=password('123456') where user='root' and Host = 'localhost';
```

123456 为新密码。

最后输入 `flush privileges;` 刷新权限

11.修改 `my.ini` 文件删除最后一句 `skip-grant-tables`

12.重启mysql即可正常使用

```sh
net stop mysql 
net start mysql
```

参考：

[windows下安装MySQL5.7.34](https://blog.csdn.net/wyj180/article/details/118881084)