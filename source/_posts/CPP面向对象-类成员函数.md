---
title: CPP面向对象-类成员函数
date: 2021-06-30 20:17:11
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

类成员函数是类的一个成员，它可以操作类的任意对象，可以访问对象中的所有成员。

`::`：范围解析运算符，也叫做作用域区分符，指明一个函数属于哪个类或一个数据属于哪个类。
`::`：可以不跟类名，表示全局数据或全局函数（即非成员函数）。

成员函数可以定义在类定义内部，或者单独使用范围解析运算符 `::` 来定义，调用成员函数是在对象上使用点运算符（`.`）。

<!--more-->
实例

```cpp
#include <iostream>
 
using namespace std;
 
class Box
{
   public:
      double length;         // 长度
      double breadth;        // 宽度
      double height;         // 高度
 
      // 成员函数声明
      double getVolume(void);
};
 
// 成员函数定义
double Box::getVolume(void)
{
    return length * breadth * height;
}
```

`定义` 在类中的成员函数缺省都是内联的，称作`内联函数`，即便没有使用 `inline` 标识符。
如果在类中未给出成员函数定义，而又想内联该函数的话，那在类外要加上 `inline`，否则就认为不是内联的。

实例：

```cpp
class A
{
    public:void Foo(int x, int y) {  } // 自动地成为内联函数
}
```

将成员函数的定义体放在类声明之中虽然能带来书写上的方便，但不是一种良好的编程风格，上例应该改成：

```cpp
// 头文件
class A
{
    public:
    void Foo(int x, int y);
}
// 定义文件
inline void A::Foo(int x, int y){} 
```

关键字 `inline` 必须与函数定义体放在一起才能使函数成为内联，仅将 `inline` 放在函数声明前面不起任何作用。

```cpp
// Foo 不能成为内联函数
inline void Foo(int x, int y); // inline 仅与函数声明放在一起
void Foo(int x, int y){}

// Foo 内联函数
void Foo(int x, int y);
inline void Foo(int x, int y) {}  // inline 与函数定义体放在一起
```

实例1

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
    double getVolume(void);
    void setLength(double len);
    void setBreadth(double bre);
    void setHeight(double hei);
};

// 成员函数定义
double Box::getVolume(void)
{
    return length * breadth * height;
}

void Box::setLength(double len)
{
    length = len;
}

void Box::setBreadth(double bre)
{
    breadth = bre;
}

void Box::setHeight(double hei)
{
    height = hei;
}

// 程序的主函数
int main()
{
    Box Box1;            // 声明 Box1，类型为 Box
    Box Box2;            // 声明 Box2，类型为 Box
    double volume = 0.0; // 用于存储体积

    // box 1 详述
    Box1.setLength(6.0);
    Box1.setBreadth(7.0);
    Box1.setHeight(5.0);

    // box 2 详述
    Box2.setLength(12.0);
    Box2.setBreadth(13.0);
    Box2.setHeight(10.0);

    // box 1 的体积
    volume = Box1.getVolume();
    cout << "Box1 的体积：" << volume << endl;

    // box 2 的体积
    volume = Box2.getVolume();
    cout << "Box2 的体积：" << volume << endl;
    return 0;
}
```

结果：

```cpp
Box1 的体积： 210
Box2 的体积： 1560
```

实例：

```cpp
int month; //全局变量
int day;
int year;

void Set(int m, int d, int y)
{
    ::year = y; //给全局变量赋值，此处可省略
    ::day = d;
    ::month = m;
}

Class Tdate
{
public:
    void Set(int m, int d, int y) //成员函数
    {
        ::Set(m, d, y); //非成员函数
    }

private:
    int month;
    int day;
    int year;
}
```








