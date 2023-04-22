---
title: Mysql-CentOS7单机安装
date: 2017-02-15 15:48:14
tags: 
  - Linux
  - MySql
  - CentOS7
  - 安装部署
categories: MySql
---

在centos7单机安装mysql.

官网：[https://www.mysql.com/cn/downloads/](https://www.mysql.com/cn/downloads/)

## 检查MySql是否安装

方式1

```linux
[root@localhost toor]# yum list installed mysql* 
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirrors.tuna.tsinghua.edu.cn
 * extras: mirrors.tuna.tsinghua.edu.cn
 * updates: mirrors.tuna.tsinghua.edu.cn
Installed Packages
mysql-community-client.x86_64             5.7.17-1.el7        @mysql57-community
mysql-community-common.x86_64             5.7.17-1.el7        @mysql57-community
mysql-community-libs.x86_64               5.7.17-1.el7        @mysql57-community
mysql-community-libs-compat.x86_64        5.7.17-1.el7        @mysql57-community
mysql-community-server.x86_64             5.7.17-1.el7        @mysql57-community
mysql57-community-release.noarch          el7-9               installed         
```

方式2

```linux
[root@localhost toor]# rpm -qa|grep -i mysql
mysql-community-libs-compat-5.7.17-1.el7.x86_64
mysql-community-libs-5.7.17-1.el7.x86_64
mysql57-community-release-el7-9.noarch
mysql-community-client-5.7.17-1.el7.x86_64
mysql-community-server-5.7.17-1.el7.x86_64
mysql-community-common-5.7.17-1.el7.x86_64

```

## 安装

注：选择root用户操作

### 下载rpm包

选择下载目录后，打开终端，输入命令，如下：

```linux
# wget http://dev.mysql.com/get/mysql57-community-release-el7-7.noarch.rpm
```

### 安装mysql57-community-release-el7-7.noarch.下载rpm包

``` linux
#rpm -ivh mysql57-community-release-el7-7.noarch.rpm
```

### 安装mysql

```linux
# yum install mysql-community-server
```

### 启动mysql服务

```linux
# service mysqld start
```

检查mysql状态

```linux
# service mysqld status
```

查看mysql版本

```linux
# mysql --version
```

### 验证mysql实例

输入命令，显示临时生成的密码：

```linux
# grep 'temporary password' /var/log/mysqld.log
```

设置root帐号（和系统root帐号不同）密码：

```linux
# mysql_secure_installation
```

### 连接到mysql server

```linux
# mysql -u root -p
```

### 使用yum更新mysql

```linux
# yum update mysql-server
```

### 开启远程访问端口

> 开启端口：

`firewall-cmd --zone=public --add-port=3306/tcp --permanent`

命令含义：

--zone #作用域

--add-port=3306/tcp #添加端口，格式为：端口/通讯协议

--permanent #永久生效，没有此参数重启后失效

> 重启防火墙:

`firewall-cmd --reload`

### 开机自动启动/取消

```linux
systemctl is-enabled iptables.service
systemctl is-enabled servicename.service #查询服务是否开机启动
systemctl enable mysqld.service #开机运行服务
systemctl disable mysqld.service #取消开机运行
systemctl start *.service #启动服务
systemctl stop *.service #停止服务
systemctl restart *.service #重启服务
systemctl reload *.service #重新加载服务配置文件
systemctl status *.service #查询服务运行状态
systemctl --failed #显示启动失败的服务
```

### GUI管理工具

* workbench(工作台)

下载地址：[https://dev.mysql.com/downloads/workbench/](https://dev.mysql.com/downloads/workbench/ "下载地址")

![配置](/img/mysql-workbench.png)

参考：

[http://blog.csdn.net/typa01_kk/article/details/49057073][removeMysql]

[http://www.tecmint.com/install-latest-mysql-on-rhel-centos-and-fedora/](http://www.tecmint.com/install-latest-mysql-on-rhel-centos-and-fedora/, "参考安装1")

[http://tecadmin.net/install-mysql-5-7-centos-rhel/](http://tecadmin.net/install-mysql-5-7-centos-rhel/, "参考安装2")

[http://blog.csdn.net/typa01_kk/article/details/49057073](http://blog.csdn.net/typa01_kk/article/details/49057073 "Mysql，检查安装，卸载")
