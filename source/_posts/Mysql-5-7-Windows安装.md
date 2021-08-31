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
选择PATH,在其后面添加: 你的mysql 安装文件下面的bin文件夹
(如: D:\Program Files\mysql-5.7\bin )(注意不要删除其他的东西)

4.新建 my.ini 文件
5.编辑my.ini文件
[mysqld]
basedir=D:\Program Files\mysql-5.7\
datadir=D:\Program Files\mysql-5.7\data\
port=3306
skip-grant-tables

#basedir表示mysql安装路径
#datadir表示mysql数据文件存储路径
#port表示mysql端口
#skip-grant-tables表示忽略密码


6.启动管理员模式下的CMD，并将路径切换至mysql下的bin目录，然后输入
mysqld –install 
7.输入 net start mysql 启动mysql服务
8.再输入 mysqld --initialize-insecure --user=mysql; 初始化数据文件
9.然后再次启动mysql 然后用命令 mysql –u root –p 进入mysql管理界面（密码可为空）
10.进入界面后更改root密码
update mysql.user set authentication_string=password('123456') where user='root' and Host = 'localhost';
123456为我的新密码 ，此处密码由您决定
最后输入flush privileges; 刷新权限
11.修改 my.ini文件删除最后一句skip-grant-tables
12.重启mysql即可正常使用
net stop mysql 
net start mysql
