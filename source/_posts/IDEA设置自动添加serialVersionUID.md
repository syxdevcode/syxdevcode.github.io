---
title: IDEA设置自动添加serialVersionUID
date: 2021-10-11 14:49:26
tags:
- Java
- IDEA
- Spring Boot
categories: 
- IDEA
---
序列化过程：序列化操作的时候系统会把当前类的serialVersionUID写入到序列化文件中，当反序列化时系统会去检测文件中的serialVersionUID，判断它是否与当前类的serialVersionUID一致，如果一致就说明序列化类的版本与当前类版本是一样的，可以反序列化成功，否则失败。

IDEA设置：

![微信截图_20211011150411.png](/img/微信截图_20211011150411.png)
