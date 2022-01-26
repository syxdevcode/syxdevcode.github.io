---
layout: sql
title: Server服务器角色与数据库角色成员身份
date: 2022-01-24 13:43:53
tags:
- 数据库
- Sql Server
categories: 
- Sql Server
---

## 服务器角色

| 角色 |  描述 |
| :----: | :----: |
| sysadmin| 执行SQL Server中的任何动作|
| serveradmin| 配置服务器设置|
| setupadmin| 安装复制和管理扩展过程|
| securityadmin| 管理登录和CREATE DATABASE的权限以及阅读审计|
| processadmin| 管理SQL Server进程|
| dbcreator| 创建和修改数据库|
| diskadmin| 管理磁盘文件|

<!--more-->
**1、sysadmin ：执行SQL Server中的任何动作：**该角色能够执行SQL Server上的任何操作。本质上，任何具有这种角色成员身份的人都是那个服务器上的sa。 在SQL Server上，Windows的Administrators组被自动映射到sysadmin角色中。这意味着服务器的Administrators组中的任何成员同时也具有对SQL数据的sa级别的访问权限。如果需要，你可以从sysadmin角色中删除Windows的administrators 组。 只有这个角色中的成员(或一个被这个角色中的成员赋予了CREATE DATABASE权限的用户)才能够创建数据库。 *sa登录一直都是sysadmin角色中的成员，并且不能从该角色中删除。

**2、serveradmin：配置服务器设置：**该角色能设置服务器范围的配置选项或关闭服务器。尽管它在范围上相当有限，但是，由该角色的成员所控制的功能对于服务器的性能会产生非常重大的影响，角色serveradmin的成员可以执行如下的动作： 向该服务器角色中添加其他登录 运行dbcc pintable命令(从而使表常驻于主内存中) 运行系统过程sp_configure(以显示或更改系统选项) 运行reconfigure选项(以更新系统过程sp_configure所做的所有改动) 使用shutdown命令关掉数据库服务器 运行系统过程sp_tableoption为用户自定义表设置选项的值

**3、setupadmin：安装复制和管理扩展过程：**角色setupadmin中的成员仅限于管理链接服务器和启动过程，可以执行如下的动作： 向该服务器角色中添加其他登录 添加、删除或配置链接的服务器 * 执行一些系统过程，如sp_serveroption

**4、securityadmin：管理登录和CREATE DATABASE的权限以及阅读审计：**角色securitypadmin中的成员用于管理登录名、读取错误日志和创建数据库许可权限的登录名，可以执行关于服务器访问和安全的所有动作。这些成员可以进行如下的系统动作： 向该服务器角色中添加其他登录 读取SQL Server的错误日志 * 运行如下的系统过程：如sp_addlinkedsrvlogin、sp_addlogin、sp_defaultdb、sp_defaultlanguage、sp_denylogin、sp_droplinkedsrvlogin、sp_droplogin、sp_grantlogin、sp_helplogins、sp_remoteoption和sp_revokelogin(所有这些系统过程都与系统安全相关。)

**5、processadmin：管理SQL Server进程：**角色processadmin中的成员用来管理SQL Server进程，如中止用户正在运行的查询。这些成员可以进行如下的动作： 向该服务器角色中添加其他登录 执行KILL命令(以取消用户进程)

**6、dbcreator：创建和修改数据库：**dbcreator中的成员仅限于创建和更改数据库，这些成员可以进行如下的动作： 向该服务器角色中添加其他登录 运行CREATE DATABASE和ALTER DATABASE语句 * 使用系统过程sp_renamedb来修改数据库的名称

**7、diskadmin：管理磁盘文件：**diskadmin的成员用来管理磁盘文件（指派给了什么文件组、附加和分离数据库，等等） ，这些成员可以进行如下的动作：： 向该服务器角色中添加其他登录 运行如下系统过程：sp_ddumpdevice和sp_dropdevice。 * 运行DISK INIT语句

## 数据库角色成员身份

**db_accessadmin：** 固定数据库角色的成员可以为 Windows 登录名、Windows 组和 SQL Server 登录名添加或删除数据库访问权限。

**db_backupoperator：** 固定数据库角色的成员可以备份该数据库。

**db_datareader：** 固定数据库角色的成员可以从所有用户表中读取所有数据。

**db_denydatawriter：** 固定数据库角色的成员不能添加、修改或删除数据库内用户表中的任何数据。

**db_ddladmin：** 固定数据库角色的成员可以在数据库中运行任何数据定义语言 (DDL) 命令。

**db_denydatareader：**固定服务器角色的成员不能读取数据库内用户表中的任何数据。

**db_denydatawriter：**固定服务器角色的成员不能添加、修改或删除数据库内用户表中的任何数据。

**db_owner：**固定数据库角色的成员可以执行数据库的所有配置和维护活动，还可以删除数据库。

**db_securityadmin：**固定数据库角色的成员可以修改角色成员身份和管理权限。向此角色中添加主体可能会导致意外的权限升级。

**public：**每个数据库用户都属于 public 数据库角色。如果未向某个用户授予或拒绝对安全对象的特定权限时，该用户将继承授予该对象的 public 角色的权限。

参考：

[SQL Server：服务器角色](https://www.cnblogs.com/rainman/p/3509626.html)

[sqlserver 数据库角色成员身份](https://blog.csdn.net/qq_39019865/article/details/90750196)