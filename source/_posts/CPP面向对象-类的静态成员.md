---
title: CPP面向对象-类的静态成员
date: 2021-07-01 16:49:58
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

使用 `static` 关键字来把类成员定义为静态的。当声明类的成员为静态时，这意味着无论创建多少个类的对象，静态成员都只有一个副本。

![cpp-static-members-2020-12-14.png](/img/cpp-static-members-2020-12-14.png)

静态成员在类的所有对象中是共享的。如果不存在其他的初始化语句，在创建第一个对象时，所有的静态数据都会被初始化为零。

我们不能把静态成员的初始化放置在类的定义中，但是可以在类的外部通过使用范围解析运算符 `::` 来重新声明静态变量从而对它进行初始化。
<!--more-->
## 静态成员变量

静态成员变量在类中仅仅是声明，没有定义，所以要在类的外面定义，实际上是给静态成员变量分配内存。如果不加定义就会报错，初始化是赋一个初始值，而定义是分配内存。

实例：

```cpp
#include <iostream>
 
using namespace std;

class Box
{
   public:
      static int objectCount;
      // 构造函数定义
      Box(double l=2.0, double b=2.0, double h=2.0)
      {
         cout <<"Constructor called." << endl;
         length = l;
         breadth = b;
         height = h;
         // 每次创建对象时增加 1
         objectCount++;
      }
      double Volume()
      {
         return length * breadth * height;
      }
   private:
      double length;     // 长度
      double breadth;    // 宽度
      double height;     // 高度
};

// 初始化类 Box 的静态成员   其实是定义并初始化的过程
int Box::objectCount = 0;
//也可这样 定义却不初始化
//int Box::objectCount;
int main(void)
{
   Box Box1(3.3, 1.2, 1.5);    // 声明 box1
   Box Box2(8.5, 6.0, 2.0);    // 声明 box2

   // 输出对象的总数
   cout << "Total objects: " << Box::objectCount << endl;

   return 0;
}
```

结果：

```cpp
Constructor called.
Constructor called.
Total objects: 2
```

## 静态成员函数

如果把函数成员声明为静态的，就可以把函数与类的任何特定对象独立开来。静态成员函数即使在类对象不存在的情况下也能被调用，静态函数只要使用类名加范围解析运算符 `::` 就可以访问。

静态成员函数只能访问静态成员数据、其他静态成员函数和类外部的其他函数。

静态成员函数有一个类范围，他们不能访问类的 `this` 指针。可以使用静态成员函数来判断类的某些对象是否已被创建。

静态成员函数与普通成员函数的区别：

* 静态成员函数没有 `this` 指针，只能访问静态成员（包括静态成员变量和静态成员函数）。
* 普通成员函数有 `this` 指针，可以访问类中的任意成员；而静态成员函数没有 `this` 指针。

```cpp
#include <iostream>

using namespace std;

class Box
{
public:
    static int objectCount;
    // 构造函数定义
    Box(double l = 2.0, double b = 2.0, double h = 2.0)
    {
        cout << "Constructor called." << endl;
        length = l;
        breadth = b;
        height = h;
        // 每次创建对象时增加 1
        objectCount++;
    }
    double Volume()
    {
        return length * breadth * height;
    }
    static int getCount()
    {
        return objectCount;
    }

private:
    double length;  // 长度
    double breadth; // 宽度
    double height;  // 高度
};

// 初始化类 Box 的静态成员
int Box::objectCount = 0;

int main(void)
{

    // 在创建对象之前输出对象的总数
    cout << "Inital Stage Count: " << Box::getCount() << endl;

    Box Box1(3.3, 1.2, 1.5); // 声明 box1
    Box Box2(8.5, 6.0, 2.0); // 声明 box2

    // 在创建对象之后输出对象的总数
    cout << "Final Stage Count: " << Box::getCount() << endl;

    return 0;
}
```

结果：

```cpp
Inital Stage Count: 0
Constructor called.
Constructor called.
Final Stage Count: 2
```

[C++ 类的静态成员](https://www.runoob.com/cplusplus/cpp-static-members.html)