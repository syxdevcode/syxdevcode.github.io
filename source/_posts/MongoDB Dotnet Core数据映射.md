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


参考：

[MongoDB via Dotnet Core数据映射详解](https://mp.weixin.qq.com/s/M3zdBDc2ci5xfL-fazTH7A)
