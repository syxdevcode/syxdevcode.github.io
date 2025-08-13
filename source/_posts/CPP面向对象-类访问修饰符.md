---
title: CPP面向对象-类访问修饰符
date: 2021-07-01 17:22:21
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

数据封装是面向对象编程的一个重要特点，它防止函数直接访问类类型的内部成员。类成员的访问限制是通过在类主体内部对各个区域标记 `public`、`private`、`protected` 来指定的。关键字 `public`、`private`、`protected` 称为访问修饰符。

一个类可以有多个 `public`、`protected` 或 `private` 标记区域。每个标记区域在下一个标记区域开始之前或者在遇到类主体结束右括号之前都是有效的。

如果继承时不显示声明是 `private`，`protected`，`public` 继承，则默认是 `private` 继承，在 `struct` 中默认 `public` 继承。

```cpp
class Base {
   public:
  // 公有成员
 
   protected:
  // 受保护成员
 
   private:
  // 私有成员
};
```
<!--more-->
## 公有（public）成员

公有成员在程序中类的外部是可访问的。可以不使用任何成员函数来设置和获取公有变量的值。

实例：

```cpp
#include <iostream>

using namespace std;

class Line
{
public:
    double length;
    void setLength(double len);
    double getLength(void);
};

// 成员函数定义
double Line::getLength(void)
{
    return length;
}

void Line::setLength(double len)
{
    length = len;
}

// 程序的主函数
int main()
{
    Line line;

    // 设置长度
    line.setLength(6.0);
    cout << "Length of line : " << line.getLength() << endl;

    // 不使用成员函数设置长度
    line.length = 10.0; // OK: 因为 length 是公有的
    cout << "Length of line : " << line.length << endl;
    return 0;
}
```

结果：

```cpp
Length of line : 6
Length of line : 10
```

## 私有（private）成员

私有成员变量或函数在类的外部是不可访问的，甚至是不可查看的。只有类和友元函数可以访问私有成员。

默认情况下，类的所有成员都是私有的。

实例

```cpp
class Box
{
    // 类的成员为私有成员
    double width;

public:
    double length;
    void setWidth(double wid);
    double getWidth(void);
};
```

实际操作中，一般会在私有区域定义数据，在公有区域定义相关的函数，以便在类的外部也可以调用这些函数。

```cpp
#include <iostream>
 
using namespace std;
 
class Box
{
   public:
      double length;
      void setWidth( double wid );
      double getWidth( void );
 
   private:
      double width;
};
 
// 成员函数定义
double Box::getWidth(void)
{
    return width ;
}
 
void Box::setWidth( double wid )
{
    width = wid;
}
 
// 程序的主函数
int main( )
{
   Box box;
 
   // 不使用成员函数设置长度
   box.length = 10.0; // OK: 因为 length 是公有的
   cout << "Length of box : " << box.length <<endl;
 
   // 不使用成员函数设置宽度
   // box.width = 10.0; // Error: 因为 width 是私有的
   box.setWidth(10.0);  // 使用成员函数设置宽度
   cout << "Width of box : " << box.getWidth() <<endl;
 
   return 0;
}
```

实例：

```cpp
Length of box : 10
Width of box : 10
```

## protected（受保护）成员

`protected`（受保护）成员变量或函数与私有成员十分相似，但有一点不同，`protected`（受保护）成员在派生类（即子类）中是可访问的。

实例：

```cpp
#include <iostream>
using namespace std;

class Box
{
protected:
    double width;
};

class SmallBox : Box // SmallBox 是派生类
{
public:
    void setSmallWidth(double wid);
    double getSmallWidth(void);
};

// 子类的成员函数
double SmallBox::getSmallWidth(void)
{
    return width;
}

void SmallBox::setSmallWidth(double wid)
{
    width = wid;
}

// 程序的主函数
int main()
{
    SmallBox box;

    // 使用成员函数设置宽度
    box.setSmallWidth(5.0);
    cout << "Width of box : " << box.getSmallWidth() << endl;

    return 0;
}
```

结果：

```cpp
Width of box : 5
```

## 继承中的特点

* 1.`public` 继承：基类 `public` 成员，`protected` 成员，`private` 成员的访问属性在派生类中分别变成：`public`, `protected`, `private`
* 2.`protected` 继承：基类 `public` 成员，`protected` 成员，`private` 成员的访问属性在派生类中分别变成：`protected`, `protected`, `private`
* 3.`private` 继承：基类 `public` 成员，`protected` 成员，`private` 成员的访问属性在派生类中分别变成：`private`, `private`, `private`

但无论哪种继承方式，上面两点都没有改变：

* 1.`private` 成员只能被本类成员（类内）和友元访问，不能被派生类访问；
* 2.`protected` 成员可以被派生类访问。

| 继承方式	| 基类的public成员	| 基类的protected成员	| 基类的private成员 | 继承引起的访问控制关系变化概括 |
|:--- | :--- |:--- |:--- |:--- |
|public继承 |	仍为public成员 |	仍为protected成员 |	不可见 |	基类的非私有成员在子类的访问属性不变 |
|protected继承 |	变为protected成员 |	变为protected成员 |	不可见 |	基类的非私有成员都为子类的保护成员 |
|private继承 |	变为private成员	| 变为private成员	| 不可见 |	基类中的非私有成员都称为子类的私有成员 |

[C++ 类访问修饰符](https://www.runoob.com/cplusplus/cpp-class-access-modifiers.html)