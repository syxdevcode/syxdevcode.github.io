---
title: CPP-成员运算符
date: 2021-06-28 17:15:53
tags:
- C语言
- CPP
- VSCode
- VS
- MingGW64
categories: CPP
---

`.`（点）运算符和 `->`（箭头）运算符用于引用类、结构和共用体的成员。

点运算符应用于实际的对象。箭头运算符与一个指向对象的指针一起使用。

实例：

```cpp
struct Employee {
  char first_name[16];
  int  age;
} emp;
```

**（.）点运算符**

下面的代码把值 `zara` 赋给对象 `emp` 的 `first_name` 成员：

```cpp
strcpy(emp.first_name, "zara");
```

**（->）箭头运算符**

如果 `p_emp` 是一个指针，指向类型为 `Employee` 的对象，则要把值 `zara` 赋给对象 emp 的 `first_name` 成员，需要编写如下代码：

```cpp
strcpy(p_emp->first_name, "zara");
```