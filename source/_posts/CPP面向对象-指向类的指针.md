---
title: CPP面向对象-指向类的指针
date: 2021-07-01 16:46:18
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

一个指向 C++ 类的指针与指向结构的指针类似，访问指向类的指针的成员，需要使用成员访问运算符 `->`，就像访问指向结构的指针一样。与所有的指针一样，必须在使用指针之前，对指针进行初始化。

<!--more-->

实例：

```cpp
#include <iostream>

using namespace std;

class Box
{
public:
    // 构造函数定义
    Box(double l = 2.0, double b = 2.0, double h = 2.0)
    {
        cout << "Constructor called." << endl;
        length = l;
        breadth = b;
        height = h;
    }
    double Volume()
    {
        return length * breadth * height;
    }

private:
    double length;  // Length of a box
    double breadth; // Breadth of a box
    double height;  // Height of a box
};

int main(void)
{
    Box Box1(3.3, 1.2, 1.5); // Declare box1
    Box Box2(8.5, 6.0, 2.0); // Declare box2
    Box *ptrBox;             // Declare pointer to a class.

    // 保存第一个对象的地址
    ptrBox = &Box1;

    // 现在尝试使用成员访问运算符来访问成员
    cout << "Volume of Box1: " << ptrBox->Volume() << endl;

    // 保存第二个对象的地址
    ptrBox = &Box2;

    // 现在尝试使用成员访问运算符来访问成员
    cout << "Volume of Box2: " << ptrBox->Volume() << endl;

    return 0;
}
```

结果：

```cpp
Constructor called.
Constructor called.
Volume of Box1: 5.94
Volume of Box2: 102
```

[C++ 指向类的指针](https://www.runoob.com/cplusplus/cpp-pointer-to-class.html)