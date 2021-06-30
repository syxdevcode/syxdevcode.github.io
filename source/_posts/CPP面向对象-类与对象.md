---
title: CPP面向对象-类与对象
date: 2021-06-30 14:59:07
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

C++ 在 C 语言的基础上增加了面向对象编程，C++ 支持面向对象程序设计。类是 C++ 的核心特性，通常被称为用户定义的类型。

类用于指定对象的形式，它包含了数据表示法和用于处理数据的方法。类中的数据和方法称为类的成员。函数在一个类中被称为类的成员。

## 类定义

类定义是以关键字 `class` 开头，后跟类的名称。类的主体是包含在一对花括号中。类定义后必须跟着一个分号或一个声明列表。

![cpp-classes-objects-2020-12-10-11.png](/img/cpp-classes-objects-2020-12-10-11.png)

实例：

```cpp
class Box
{
   public:
      double length;   // 盒子的长度
      double breadth;  // 盒子的宽度
      double height;   // 盒子的高度
};
```

关键字 `public`，`private` 或 `protected` 确定了类成员的访问属性。

## 定义对象

声明类的对象，与声明基本类型的变量一样。

```cpp
Box Box1;          // 声明 Box1，类型为 Box
Box Box2;          // 声明 Box2，类型为 Box
```

## 访问数据成员

类的对象的公共数据成员可以使用直接成员访问运算符 `.` 来访问。

![cpp-classes-objects-2020-12-10-11-2.png](/img/cpp-classes-objects-2020-12-10-11-2.png)

实例

```cpp
#include <iostream>

using namespace std;

class Box
{
public:
    double length;  // 长度
    double breadth; // 宽度
    double height;  // 高度
    // 成员函数声明
    double get(void);
    void set(double len, double bre, double hei);
};
// 成员函数定义
double Box::get(void)
{
    return length * breadth * height;
}

void Box::set(double len, double bre, double hei)
{
    length = len;
    breadth = bre;
    height = hei;
}
int main()
{
    Box Box1;            // 声明 Box1，类型为 Box
    Box Box2;            // 声明 Box2，类型为 Box
    Box Box3;            // 声明 Box3，类型为 Box
    double volume = 0.0; // 用于存储体积

    // box 1 详述
    Box1.height = 5.0;
    Box1.length = 6.0;
    Box1.breadth = 7.0;

    // box 2 详述
    Box2.height = 10.0;
    Box2.length = 12.0;
    Box2.breadth = 13.0;

    // box 1 的体积
    volume = Box1.height * Box1.length * Box1.breadth;
    cout << "Box1 的体积：" << volume << endl;

    // box 2 的体积
    volume = Box2.height * Box2.length * Box2.breadth;
    cout << "Box2 的体积：" << volume << endl;

    // box 3 详述
    Box3.set(16.0, 8.0, 12.0);
    volume = Box3.get();
    cout << "Box3 的体积：" << volume << endl;
    return 0;
}
```

结果：

```cpp
Box1 的体积：210
Box2 的体积：1560
Box3 的体积：1536
```

注意：私有的成员和受保护的成员不能使用直接成员访问运算符 `.` 来直接访问。

[C++ 类&对象](https://www.runoob.com/cplusplus/cpp-classes-objects.html)