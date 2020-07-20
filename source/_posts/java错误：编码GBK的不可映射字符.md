---
title: java错误：编码GBK的不可映射字符
date: 2020-07-20 13:43:37
tags:
- java
- idea
categories: 
- java
---

当Java源代码中包含中文字符时，我们在用javac编译时会出现“错误：编码GBK的不可映射字符”。

由于JDK是国际版的，我们在用javac编译时，编译程序首先会获得我们操作系统默认采用的编码格式（GBK），然后JDK就把Java源文件从GBK编码格式转换为Java内部默认的Unicode格式放入内存中，然后javac把转换后的Unicode格式的文件编译成class类文件，此时，class文件是Unicode编码的，它暂存在内存中，紧接着，JDK将此以Unicode格式编码的class文件保存到操作系统中形成我们见到的class文件。当我们不加设置就编译时，相当于使用了参数：`javac -encoding GBK Test.java`，就会出现不兼容的情况。

使用-encoding参数指明编码方式：`javac -encoding UTF-8 Test.java`，就可以了。

转载：[错误：编码GBK的不可映射字符](https://www.cnblogs.com/lucky-zhangcd/p/8409810.html)