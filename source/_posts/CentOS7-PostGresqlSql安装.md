---
title: 'CentOS7-PostGresqlSql9.6安装'
date: 2017-09-15 14:35:18
tags:
- Linux
- CentOS7
- PostGresqlSql
- 安装部署
categories:
- PostGresqlSql
---
## 下载安装

[官网文档](https://www.postgresql.org/download/linux/redhat/ "链接")

注: root账户安装

### 添加RPM

```` linux
yum install https://download.postgresql.org/pub/repos/yum/9.6/redhat/rhel-7-x86_64/pgdg-centos96-9.6-3.noarch.rpm
````

### 安装PostgreSQL

````linux
yum install postgresql96         ##客户端包##
yum install postgresql96-server  ##服务端包##
````

安装过程出现问题，可能会导致初始化数据库错误错误，如下：

解决方法：重新执行命令 `yum install postgresql96-server`

````linux
[root@iz2zeii3e4srrxcoy1gbubz ~]# /usr/pgsql-9.6/bin/postgresql96-setup initdb
-bash: /usr/pgsql-9.6/bin/postgresql96-setup: No such file or directory
````

### 初始化数据库,允许自动启动

````linux
/usr/pgsql-9.6/bin/postgresql96-setup initdb
systemctl enable postgresql-9.6
systemctl start postgresql-9.6
````

## 配置

PostgreSQL 安装完成后，会建立一下‘postgres’用户，用于执行PostgreSQL，数据库中也会建立一个'postgres'用户，默认密码为自动生成，需要在系统中改一下。

### 修改默认密码

````linux
su - postgres  ##切换用户，执行后提示符会变为 '-bash-4.2$'
psql -U postgres ##登录数据库，执行后提示符变为 'postgres=#'
ALTER USER postgres WITH PASSWORD 'abc123' ## 设置postgres用户密码
\q       ##退出数据库
exit     ##退出
````

### 开启远程访问

```linux
vi /var/lib/pgsql/9.6/data/postgresql.conf
```

 修改#listen_addresses = 'localhost' 为  listen_addresses='*'

 当然，此处‘*’也可以改为任何你想开放的服务器IP

### 信任远程连接

```linux
 vi /var/lib/pgsql/9.6/data/pg_hba.conf
```

修改如下内容，信任指定服务器连接

```linux
# IPv4 local connections:
host    all       all      127.0.0.1/32      trust
host    all       all      10.211.55.6/32（需要连接的服务器IP）  trust
```

### 打开防火墙

CentOS 防火墙中内置了PostgreSQL服务，配置文件位置在

/usr/lib/firewalld/services/postgresql.xml，

我们只需以服务方式将PostgreSQL服务开放即可。

```linux
firewall-cmd --add-service=postgresql --permanent  开放postgresql服务
firewall-cmd --reload  重载防火墙
```

### 重启服务

``` linux
systemctl restart postgresql-9.6.service
```

## 开发备注

本地服务器连接数据库出现以下错误：

```linux
OperationalError: FATAL: Ident authentication failed for user “dbuser”
```

出现这个错误的原因还是在于上面pg_hba.conf 文件的设置，Debian系（包括ubuntu）默认的pg_hba.conf 文件对于

localhost本地机器的数据库访问方式是ident，它指的是只有Linux shell用户通过同名的postgreSQL 用户才能访问，

也就是pg超级用户postgres 只能由linux 用户postgres 登录后操作。

“postgres”的问题，有两种解决方法：

* 在执行$ python manage.py shell之前先$su postgres 切换为postgres 用户

* 修改pg_hba.conf 的客户端访问设置，将laocal 的访问由ident 改为trust，如：

```linux
# TYPE DATABASE USER CIDR-ADDRESS METHOD
local all all trust
```

修改完pg_hba.conf设置记得重启pg。安装了pg_ctl 也可以用pg_ctl reload。

参考：

[官网文档](https://www.postgresql.org/download/linux/redhat/ "链接")

[http://www.jianshu.com/p/7e95fd0bc91a](http://www.jianshu.com/p/7e95fd0bc91a "链接")

[http://www.cnblogs.com/mchina/archive/2012/06/06/2539003.html](http://www.cnblogs.com/mchina/archive/2012/06/06/2539003.html "链接")

[http://blog.csdn.net/wang1144/article/details/8986479](http://blog.csdn.net/wang1144/article/details/8986479 "本地连接错误")
