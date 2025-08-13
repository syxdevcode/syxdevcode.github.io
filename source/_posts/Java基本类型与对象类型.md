---
title: Java基本类型与对象类型
date: 2021-09-27 09:06:47
tags:
- Java
categories: 
- Java
---

## Java两种数据类型

**基本类型：**

```java
byte(8)
short(16)
int(32)
long(64)
float(32)
double(64)
char(16)
boolean(1)
```

**对象类型：**

```java
Byte
Short
Integer
Long
Float
Double
Character
Boolean
```

对象类型分别是基本类型的包装类，例如 `Byte` 是 `byte` 的包装类.

## 用途

Java语言是一个面向对象的语言，但是Java中的基本数据类型却是不面向对象的，这在实际使用时存在很多的不便，为了解决这个不足，在设计类时为每个基本数据类型设计了一个对应的类进行代表，这样八个和基本数据类型对应的类统称为`包装类（Wrapper Class）`。

对于包装类说，这些类的用途主要包含两种：

* 作为和基本数据类型对应的类类型存在，方便涉及到对象的操作。

* 包含每种基本数据类型的相关属性如最大值、最小值等，以及相关的操作方法。
