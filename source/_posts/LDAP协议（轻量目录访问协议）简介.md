---
title: LDAP协议（轻量目录访问协议）简介
date: 2022-04-17 09:43:59
tags:
- LDAP协议
categories:
- 计算机基础
---


## 简介

LDAP 的全称是 Lightweight Directory Access Protocol，「轻量目录访问协议」，它是基于X.500标准的轻量级目录访问协议。
目录是一个为查询、浏览和搜索而优化的数据库，它成树状结构组织数据，类似文件目录一样。
目录数据库和关系数据库不同，它有优异的读性能，但写性能差，并且没有事务处理、回滚等复杂功能，不适于存储修改频繁的数据。
LDAP 「是一个协议」，约定了 Client 与 Server 之间的信息交互格式、使用的端口号、认证方式等内容。

## LDAP的基本模型

### 目录树概念

* 1. 目录树：在一个目录服务系统中，整个目录信息集可以表示为一个目录信息树，树中的每个节点是一个条目。
* 2. 条目：每个条目就是一条记录，每个条目有自己的唯一可区别的名称（DN）。
* 3. 对象类：与某个实体类型对应的一组属性，对象类是可以继承的，这样父类的必须属性也会被继承下来。
* 4. 属性：描述条目的某个方面的信息，一个属性由一个属性类型和一个或多个属性值组成，属性有必须属性和非必须属性。

<!--more-->

### 名词解释

**DIT（Directory Information Tree）目录信息树**
LDAP 目录服务器将信息以树形的方式组织，每一项都可以包含 0 个或多个子项。这样的结构叫做目录信息树。

**Entry 项**
在用户目录中，看到的每一行，都可以叫做一项，不论是叶子节点还是中间的节点。

![v2-7d18c62141bea796afdc66713ff47765_720w.jpg](/img/v2-7d18c62141bea796afdc66713ff47765_720w.jpg)

项包含一个 DN，一些属性，一些对象类。

![v2-ebca130763dabaafaf41c92eef0406f5_720w.jpg](/img/v2-ebca130763dabaafaf41c92eef0406f5_720w.jpg)

**Root DSE（Root DSA-specific entry）根节点项**
每个 LDAP 服务器必须对外暴露一个特殊的「项」，叫做 root DSE，这个项的 DN 是空字符串。这个项是根节点，描述了 LDAP 服务器自身的信息和能力。例如你可以在下图看到 LDAP 服务器支持的功能，LDAP 协议版本等信息。

![v2-7cb7860f9a897cc78ebb68b09fc9c58e_720w.jpg](/img/v2-7cb7860f9a897cc78ebb68b09fc9c58e_720w.jpg)

**dn（Distinguished Name）分辨名**
dn 如下图白色方框中的内容，「分辨名」用于唯一标识一个「项」，以及他在目录信息树中的位置。可以和文件系统中文件路径类比。类似于关系型数据库中的主键。dn 字符串从左向右，各组成部分依次向树根靠近。

![v2-528f3a7814a9e63ba6883d36eca3dd6a_720w.jpg](/img/v2-528f3a7814a9e63ba6883d36eca3dd6a_720w.jpg)

**rdn（Relative Distinguished Name）相对分辨名**
Rdn 就是「键值对」，如下图黄色方框中的内容。dn 由若干个 rdn 组成，以逗号分隔。

![v2-528f3a7814a9e63ba6883d36eca3dd6a_720w.jpg](/img/v2-528f3a7814a9e63ba6883d36eca3dd6a_720w.jpg)

**dc（Domain Component）（域名组成）**
example.com变成dc=example,dc=com（一条记录的所属位置）

**o（Organization）组织机构、公司**
在 dn 中可能会包含 o=公司 这样的组成部分，这里的 o 指代组织机构。

**uid (User Id)**
用户ID songtao.xu（一条记录的ID）

**ou（Organization Unit）组织单元、部门**
在 dn 中可能会包含 ou=某某部门 这样的组成部分，这里的 ou 指代组织单元。

**Object Classes**
每个「项」里面包含若干个 Object Classes，「Object Class」 指定了本项中必须、可能包含的「属性」，相当于 MySQL 中的建表语句。包含某个 Object Class 的项，必须满足 Object Class 中约定的规范。如下图，「person objectClass」 中规定了 cn 和 sn 属性必须存在，在这一「项」的属性中，你可以看到 cn 和 sn 被加粗显示了。

![v2-ebca130763dabaafaf41c92eef0406f5_720w(1).jpg](/img/v2-ebca130763dabaafaf41c92eef0406f5_720w.jpg)

下图是 person Object class 的定义，规定了拥有此 Object class 的「项」需要拥有的属性：

![v2-dd88c919ea49ff23bd7ea5e0547833e8_720w.jpg](/img/v2-dd88c919ea49ff23bd7ea5e0547833e8_720w.jpg)

## LDAP的主要产品

LDAP仅仅是一个访问协议，数据存储各厂商的详细表：

* Microsoft

产品：Microsoft Active Directory

基于WINDOWS系统用户，对大数据量处理速度一般，但维护容易，生态圈大，管理相对简单。

* SUN

产品：SUNONE Directory Server

基于文本数据库的存储，速度快。

* IBM

产品：IBM Directory Server

基于DB2 的的数据库，速度一般。

* Novell

产品：Novell Directory Server

基于文本数据库的存储，速度快, 不常用到。

* Opensource

产品：Opensource

OpenLDAP 开源的项目，速度很快，但是非主 流应用。

## LDAP协议解决的问题

### 用户服务

管理用户的域账号、用户信息、企业通信录（与电子邮箱系统集成）、用户组管理、用户身份认证、用户授权管理、按需实施组管理策略等。在 Windows 下，有组策略管理器，如果启用域用户认证，那么这些组策略可以统一管理，方便地限制用户的权限。

### 计算机管理

管理服务器及客户端计算机账户、所有服务器及客户端计算机加入域管理并按需实施组策略，甚至可以控制计算机禁止修改壁纸。（什么？给电脑重装系统就能解除限制？那么所有域上的资源都会无法访问了。）

### 资源管理

管理打印机、文件共享服务、网络资源等实施组策略。

### 应用系统的支持

对于电子邮件（Exchange）、在线及时通讯（Lync）、企业信息管理（SharePoint）、微软 CRM，ERP 等业务系统提供数据认证（身份认证、数据集成、组织规则等）。这里不单是微软产品的集成，其它的业务系统根据公用接口的方式一样可以嵌入进来。

参考：

[LDAP概念和原理介绍](https://www.cnblogs.com/wilburxu/p/9174353.html)

[LDAP 协议入门（轻量目录访问协议）](https://zhuanlan.zhihu.com/p/147768058)