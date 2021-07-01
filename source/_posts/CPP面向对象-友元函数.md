---
title: CPP面向对象-友元函数
date: 2021-07-01 15:50:18
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

类的友元函数是定义在类外部，但有权访问类的所有私有（`private`）成员和保护（`protected`）成员。尽管友元函数的原型有在类的定义中出现过，但是友元函数并不是成员函数。

友元可以是一个函数，该函数被称为`友元函数`；
友元也可以是一个类，该类被称为`友元类`，在这种情况下，整个类及其所有成员都是友元。

使用关键字 `friend`，定义 `友元函数` 与 `友元类`。

```cpp
// 友元函数
class Box
{
   double width;
public:
   double length;
   friend void printWidth( Box box );
   void setWidth( double wid );
};

// 友元类
friend class ClassTwo;
```
<!--more-->

**友元函数的使用:**

因为友元函数没有this指针，则参数要有三种情况： 

* 要访问非 `static` 成员时，需要对象做参数；
* 要访问 `static` 成员或全局变量时，则不需要对象做参数；
* 如果做参数的对象是全局对象，则不需要对象做参数，可以直接调用友元函数，不需要通过对象或指针。

实例：

友元类，友元函数的使用：

```cpp
#include <iostream>

using namespace std;

class Box
{
    double width;

public:
    friend void printWidth(Box box);
    friend class BigBox;
    void setWidth(double wid);
};

class BigBox
{
public:
    void Print(int width, Box &box)
    {
        // BigBox是Box的友元类，它可以直接访问Box类的任何成员
        box.setWidth(width);
        cout << "Width of box : " << box.width << endl;
    }
};

// 成员函数定义
void Box::setWidth(double wid)
{
    width = wid;
}

// 请注意：printWidth() 不是任何类的成员函数
void printWidth(Box box)
{
    /* 因为 printWidth() 是 Box 的友元，它可以直接访问该类的任何成员 */
    cout << "Width of box : " << box.width << endl;
}

// 程序的主函数
int main()
{
    Box box;
    BigBox big;

    // 使用成员函数设置宽度
    box.setWidth(10.0);

    // 使用友元函数输出宽度
    printWidth(box);

    // 使用友元类中的方法设置宽度
    big.Print(20, box);

    getchar();
    return 0;
}
```

结果：

```cpp
Width of box : 10
Width of box : 20
```