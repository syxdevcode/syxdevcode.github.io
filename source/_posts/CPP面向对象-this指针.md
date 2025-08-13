---
title: CPP面向对象-this指针
date: 2021-07-01 16:10:40
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

在 C++ 中，每一个对象都能通过 `this` 指针来访问自己的地址。`this` 指针是所有成员函数的隐含参数。因此，在成员函数内部，它可以用来指向调用对象。

友元函数没有 `this` 指针，因为友元不是类的成员。只有成员函数才有 `this` 指针。
<!--more-->
**引入this：**

当我们调用成员函数时，实际上是替某个对象调用它。

成员函数通过一个名为 `this` 的额外隐式参数来访问调用它的那个对象，当我们调用一个成员函数时，用请求该函数的对象地址初始化 `this`。

例如，如果调用 `total.isbn()` 则编译器负责把 `total` 的地址传递给 `isbn` 的隐式形参 `this`，可以等价地认为编译器将该调用重写成了以下形式：

```cpp
//伪代码，用于说明调用成员函数的实际执行过程
Sales_data::isbn(&total)
```

其中，调用 `Sales_data` 的 `isbn` 成员时传入了 `total` 的地址。

在成员函数内部，我们可以直接使用调用该函数的对象的成员，而无须通过成员访问运算符来做到这一点，因为 `this` 所指的正是这个对象。

任何对类成员的直接访问都被看作是对 `this` 的隐式引用，也就是说，当 `isbn` 使用 `bookNo` 时，它隐式地使用 `this` 指向的成员，就像我们书写了 `this->bookNo` 一样。

对于我们来说，`this` 形参是隐式定义的。实际上，任何自定义名为 `this` 的参数或变量的行为都是非法的。我们可以在成员函数体内部使用 `this`，因此尽管没有必要，我们还是能把 `isbn` 定义成如下形式：

```cpp
std::string isbn() const { return this->bookNo; }
```

因为 `this` 的目的总是指向 `这个` 对象，所以 `this` 是一个常量指针，我们不允许改变 `this` 中保存的地址。

实例1：

```cpp
#include <iostream>

using namespace std;

class Box
{
public:
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
    int compare(Box box)
    {
        int t = this->Volume() > box.Volume();
        cout << t << endl;
        return t;
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

    if (Box1.compare(Box2))
    {
        cout << "Box2 is smaller than Box1" << endl;
    }
    else
    {
        cout << "Box2 is equal to or larger than Box1" << endl;
    }
    return 0;
}
```

结果：

```cpp
Constructor called.
Constructor called.
0
Box2 is equal to or larger than Box1
```

实例2：

`this` 指针的类型可理解为 `Box*`。

```cpp
#include <iostream>
using namespace std;

class Box
{
public:
    Box() { ; }
    ~Box() { ; }
    Box *get_address() //得到this的地址
    {
        return this;
    }
};

int main()
{

    Box box1;
    Box box2;
    // Box* 定义指针p接受对象box的get_address()成员函数的返回值，并打印

    Box *p = box1.get_address();
    cout << p << endl;

    p = box2.get_address();
    cout << p << endl;

    return 0;
}
```

结果：

```cpp
0x61fe07
0x61fe06
```

[C++ this 指针](https://www.runoob.com/cplusplus/cpp-this-pointer.html)