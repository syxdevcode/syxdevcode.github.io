---
title: CPP面向对象-构造函数与析构函数
date: 2021-07-01 14:05:56
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

## 类构造函数

类的构造函数是类的一种特殊的成员函数，它会在每次创建类的新对象时执行。

构造函数的名称与类的名称是完全相同的，并且不会返回任何类型，也不会返回 `void`。构造函数可用于为某些成员变量设置初始值。

一个类内可以有多个构造函数，可以是一般类型的，也可以是带参数的，相当于重载构造函数，但是析构函数只能有一个。

<!--more-->
实例：

```cpp
#include <iostream>

using namespace std;

class Line
{
public:
    void setLength(double len);
    double getLength(void);
    Line(); // 这是构造函数

private:
    double length;
};

// 成员函数定义，包括构造函数
Line::Line(void)
{
    cout << "Object is being created" << endl;
}

void Line::setLength(double len)
{
    length = len;
}

double Line::getLength(void)
{
    return length;
}
// 程序的主函数
int main()
{
    Line line;

    // 设置长度
    line.setLength(6.0);
    cout << "Length of line : " << line.getLength() << endl;

    return 0;
}
```

结果：

```cpp
Object is being created
Length of line : 6
```

## 带参数的构造函数

默认的构造函数没有任何参数，但如果需要，构造函数也可以带有参数，这样在创建对象时就会给对象赋初始值。

```cpp
#include <iostream>

using namespace std;

class Line
{
public:
    void setLength(double len);
    double getLength(void);
    Line(double len); // 这是构造函数

private:
    double length;
};

// 成员函数定义，包括构造函数
Line::Line(double len)
{
    cout << "Object is being created, length = " << len << endl;
    length = len;
}

void Line::setLength(double len)
{
    length = len;
}

double Line::getLength(void)
{
    return length;
}
// 程序的主函数
int main()
{
    Line line(10.0);

    // 获取默认设置的长度
    cout << "Length of line : " << line.getLength() << endl;
    // 再次设置长度
    line.setLength(6.0);
    cout << "Length of line : " << line.getLength() << endl;

    return 0;
}
```

结果：

```cpp
Object is being created, length = 10
Length of line : 10
Length of line : 6
```

## 使用初始化列表来初始化字段

使用初始化列表来初始化字段：

```cpp
Line::Line( double len): length(len)
{
    cout << "Object is being created, length = " << len << endl;
}
```

同于如下语法：

```cpp
Line::Line( double len)
{
    length = len;
    cout << "Object is being created, length = " << len << endl;
}
```

假设有一个类 C，具有多个字段 X、Y、Z 等需要进行初始化，同理地，可以使用上面的语法，只需要在不同的字段使用逗号进行分隔，如下所示：

```cpp
C::C( double a, double b, double c): X(a), Y(b), Z(c)
{
  ....
}
```


初始化列表的成员初始化顺序:

C++ 初始化类成员时，是按照声明的顺序初始化的，而不是按照出现在初始化列表中的顺序。

```cpp
class CMyClass {
    CMyClass(int x, int y);
    int m_x;
    int m_y;
};

CMyClass::CMyClass(int x, int y) : m_y(y), m_x(m_y)
{
};
```

编译器先初始化 `m_x`，然后是 `m_y`,，因为它们是按这样的顺序声明的。结果是 `m_x` 将有一个不可预测的值。

有两种方法避免它，
一个是总是按照你希望它们被初始化的顺序声明成员，
第二个是，如果你决定使用初始化列表，总是按照它们声明的顺序罗列这些成员。这将有助于消除混淆。

## 析构函数

类的析构函数是类的一种特殊的成员函数，它会在每次删除所创建的对象时执行。

析构函数的名称与类的名称是完全相同的，只是在前面加了个波浪号（`~`）作为前缀，它不会返回任何值，也不能带有任何参数。析构函数有助于在跳出程序（比如关闭文件、释放内存等）前释放资源。

实例

```cpp
#include <iostream>
 
using namespace std;
 
class Line
{
   public:
      void setLength( double len );
      double getLength( void );
      Line();   // 这是构造函数声明
      ~Line();  // 这是析构函数声明
 
   private:
      double length;
};
 
// 成员函数定义，包括构造函数
Line::Line(void)
{
    cout << "Object is being created" << endl;
}
Line::~Line(void)
{
    cout << "Object is being deleted" << endl;
}
 
void Line::setLength( double len )
{
    length = len;
}
 
double Line::getLength( void )
{
    return length;
}
// 程序的主函数
int main( )
{
   Line line;
 
   // 设置长度
   line.setLength(6.0); 
   cout << "Length of line : " << line.getLength() <<endl;
 
   return 0;
}
```

结果：

```cpp
Object is being created
Length of line : 6
Object is being deleted
```





