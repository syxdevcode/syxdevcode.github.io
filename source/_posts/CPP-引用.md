---
title: CPP-引用
date: 2021-06-30 10:31:03
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

## 引用

引用变量是一个别名，也就是说，它是某个已存在变量的另一个名字。一旦把引用初始化为某个变量，就可以使用该引用名称或变量名称来指向变量。

## 引用vs指针

三个主要的不同：

* 不存在空引用。引用必须连接到一块合法的内存。
* 一旦引用被初始化为一个对象，就不能被指向到另一个对象。指针可以在任何时候指向到另一个对象。
* 引用必须在创建时被初始化。指针可以在任何时间被初始化。

## 创建引用

`&` 读作引用。

实例：

```cpp
int i = 17;

// 读作 "r 是一个初始化为 i 的整型引用"。
int&  r = i;

// 读作 "s 是一个初始化为 d 的 double 型引用"
double& s = d;
```

`int& r = i;` 和 `int r = i;` 不同之处应该是内存的分配吧，后者会再开辟一个内存空间。

<!--more-->
验证实例：

```cpp
#include <iostream>

using namespace std;

int main()
{
    int i;
    int &r = i;
    i = 5;
    cout << "Value of i : " << i << endl;
    cout << "Value of i reference : " << r << endl;
    cout << "Addr of i: " << &i << endl;
    cout << "Addr of r: " << &r << endl;

    int x;
    int y = x;
    x = 6;
    cout << "Value of x : " << x << endl;
    cout << "Value of y : " << y << endl;
    cout << "Addr of x: " << &x << endl;
    cout << "Addr of y: " << &y << endl;

    return 0;
}
```

结果：

```cpp
Value of i : 5
Value of i reference : 5
Addr of i: 0x61fe14
Addr of r: 0x61fe14
Value of x : 6
Value of y : 6497456
Addr of x: 0x61fe10
Addr of y: 0x61fe0c
```

实例：

```cpp
#include <iostream>

using namespace std;

int main()
{
    // 声明简单的变量
    int i;
    double d;

    // 声明引用变量
    int &r = i;
    double &s = d;

    i = 5;
    cout << "Value of i : " << i << endl;
    cout << "Value of i reference : " << r << endl;

    d = 11.7;
    cout << "Value of d : " << d << endl;
    cout << "Value of d reference : " << s << endl;

    return 0;
}
```

结果：

```cpp
Value of i : 5
Value of i reference : 5
Value of d : 11.7
Value of d reference : 11.7
```

## 把引用作为函数参数

C++之所以增加引用类型， 主要是把它作为函数参数，以扩充函数传递数据的功能。

C++ 函数传参：

* (1)将变量名作为实参和形参。这时传给形参的是变量的值，传递是单向的。如果在执行函数期间形参的值发生变化，并不传回给实参。因为在调用函数时，形参和实参不是同一个存储单元。// 同 c
* (2) 传递变量的指针。形参是指针变量，实参是一个变量的地址，调用函数时，形参(指针变量)指向实参变量单元。这种通过形参指针可以改变实参的值。// 同 c
* (3) C++提供了 传递变量的引用。形参是引用变量，和实参是一个变量，调用函数时，形参(引用变量)指向实参变量单元。这种通过形参引用可以改变实参的值。

实例：

```cpp
#include <iostream>
using namespace std;

// 函数声明
void swap(int &x, int &y);

int main()
{
    // 局部变量声明
    int a = 100;
    int b = 200;

    cout << "交换前，a 的值：" << a << endl;
    cout << "交换前，b 的值：" << b << endl;

    /* 调用函数来交换值 */
    swap(a, b);

    cout << "交换后，a 的值：" << a << endl;
    cout << "交换后，b 的值：" << b << endl;

    return 0;
}

// 函数定义
void swap(int &x, int &y)
{
    int temp;
    temp = x; /* 保存地址 x 的值 */
    x = y;    /* 把 y 赋值给 x */
    y = temp; /* 把 x 赋值给 y  */

    return;
}
```

结果：

```cpp
交换前，a 的值： 100
交换前，b 的值： 200
交换后，a 的值： 200
交换后，b 的值： 100
```

## 把引用作为函数返回值

通过使用引用来替代指针，会使 C++ 程序更容易阅读和维护。C++ 函数可以返回一个引用，方式与返回一个指针类似。

当函数返回一个引用时，则返回一个指向返回值的隐式指针。这样，函数就可以放在赋值语句的左边。

引用作为返回值：
* （1）以引用返回函数值，定义函数时需要在函数名前加 `&`
* （2）用引用返回一个函数值的最大好处是，在内存中不产生被返回值的副本。

必须遵守以下规则：

* （1）不能返回局部变量的引用。主要原因是局部变量会在函数返回后被销毁，因此被返回的引用就成为了 `无所指`的引用，程序会进入未知状态。
* （2）不能返回函数内部 `new` 分配的内存的引用。虽然不存在局部变量的被动销毁问题，可对于这种情况（返回函数内部`new`分配内存的引用），又面临其它尴尬局面。例如，被函数返回的引用只是作为一 个临时变量出现，而没有被赋予一个实际的变量，那么这个引用所指向的空间（由`new`分配）就无法释放，造成 `memory leak`。
* （3）可以返回类成员的引用，但最好是 `const`。主要原因是当对象的属性是与某种业务规则（`business rule`）相关联的时候，其赋值常常与某些其它属性或者对象的状态有关，因此有必要将赋值操作封装在一个业务规则当中。如果其它对象可以获得该属性的非常量引用（或指针），那么对该属性的单纯赋值就会破坏业务规则的完整性。

```cpp
#include <iostream>

using namespace std;

double vals[] = {10.1, 12.6, 33.1, 24.1, 50.0};

double &setValues(int i)
{
    double &ref = vals[i];
    return ref; // 返回第 i 个元素的引用，ref 是一个引用变量，ref 引用 vals[i]，最后再返回 shit。
}

// 要调用上面定义函数的主函数
int main()
{

    cout << "改变前的值" << endl;
    for (int i = 0; i < 5; i++)
    {
        cout << "vals[" << i << "] = ";
        cout << vals[i] << endl;
    }

    setValues(1) = 20.23; // 改变第 2 个元素
    setValues(3) = 70.8;  // 改变第 4 个元素

    cout << "改变后的值" << endl;
    for (int i = 0; i < 5; i++)
    {
        cout << "vals[" << i << "] = ";
        cout << vals[i] << endl;
    }
    return 0;
}
```

结果：

```cpp
改变前的值
vals[0] = 10.1
vals[1] = 12.6
vals[2] = 33.1
vals[3] = 24.1
vals[4] = 50
改变后的值
vals[0] = 10.1
vals[1] = 20.23
vals[2] = 33.1
vals[3] = 70.8
vals[4] = 50
```

当返回一个引用时，要注意被引用的对象不能超出作用域。所以返回一个对局部变量的引用是不合法的，但是，可以返回一个对静态变量的引用。

```cpp
int& func() {
   int q;
   //! return q; // 在编译时发生错误
   static int x;
   return x;     // 安全，x 在函数作用域外依然是有效的
}
```

[C++ 引用](https://www.runoob.com/cplusplus/cpp-references.html)