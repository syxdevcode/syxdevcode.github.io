---
title: MongoDB Dotnet Core数据映射
date: 2020-09-28 13:56:36
tags:
- Linux
- NoSQL
- CentOS7
- Windows
- MongoDB
- 安装部署
categories:
- MongoDB
---

## 简介

作为一个数据库，基本的操作就是CRUD。MongoDB的CRUD，不使用SQL来写，而是提供了更简单的方式。

方式一、BsonDocument方式

BsonDocument方式，适合能熟练使用MongoDB Shell的开发者。MongoDB Driver提供了完全覆盖Shell命令的各种方式，来处理用户的CRUD操作。

这种方法自由度很高，可以在不需要知道完整数据集结构的情况下，完成数据库的CRUD操作。

方式二、数据映射方式

数据映射是最常用的一种方式。准备好需要处理的数据类，直接把数据类映射到MongoDB，并对数据集进行CRUD操作。

## 字段映射

### ID字段

`MongoDB` 数据集中存放的数据，称之为文档（`Document`）。每个文档在存放时，都需要有一个ID，而这个 `ID` 的名称，固定叫 `_id`。

当我们建立映射时，如果给出 `_id` 字段，则 `MongoDB` 会采用这个ID做为这个文档的ID，如果不给出，`MongoDB` 会自动添加一个 `_id` 字段。 


在使用上是完全一样的。唯一的区别是，如果映射类中不写 `_id`，则 `MongoDB` 自动添加 `_id` 时，会用 `ObjectId` 作为这个字段的数据类型。

`ObjectId` 是一个全局唯一的数据。

`MongoDB` 允许使用其它类型的数据作为ID，例如：`string`，`int`，`long`，`GUID` 等，但这就需要你自己去保证这些数据不超限并且唯一。

## 2. 简单字段



参考：

[MongoDB via Dotnet Core数据映射详解](https://mp.weixin.qq.com/s/M3zdBDc2ci5xfL-fazTH7A)
