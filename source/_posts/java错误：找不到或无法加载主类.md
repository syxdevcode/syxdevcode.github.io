---
title: java错误：找不到或无法加载主类
date: 2020-07-20 13:46:06
tags:
- Java
- IDEA
categories: 
- Java
---

1.编译的时候后缀带了.class：把后缀去掉；
2.删除java文件中的package包；（不推荐）
2.当java文件带了package包，但是还在java文件所在目录运行，这种情况会报错；

正确做法：返回包的前一目录进行运行；

正确示例：

```java
PS E:\java\javademo\src> javac -encoding UTF-8 ServerSocketTest/GreetingServer.java
PS E:\java\javademo\src> java ServerSocketTest/GreetingServer
```