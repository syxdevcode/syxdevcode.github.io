---
title: CPP面向对象-多态
date: 2021-07-02 13:59:43
tags:
- CPP
- CPP面向对象
- VSCode
- MingGW64
categories: CPP
---

**多态**：有多个不同的类，都带有同一个名称但具有不同实现的函数，函数的参数甚至可以是相同的。

C++多态意味着调用成员函数时，会根据调用函数的对象的类型来执行不同的函数；

形成多态必须具备三个条件：

* 1、必须存在继承关系；
* 2、继承关系必须有同名虚函数（**其中虚函数是在基类中使用关键字Virtual声明的函数，在派生类中重新定义基类中定义的虚函数时，会告诉编译器不要静态链接到该函数**）；
* 3、存在基类类型的指针或者引用，通过该指针或引用调用虚函数；

<!--more-->
实例：

**静态多态，或静态链接**：即函数调用在程序执行前就准备好。也被称为`早绑定`，因为 `area()` 函数在程序编译期间就已经设置好了。

```cpp
#include <iostream>
using namespace std;

class Shape
{
protected:
    int width, height;

public:
    Shape(int a = 0, int b = 0)
    {
        width = a;
        height = b;
    }
    int area()
    {
        cout << "Parent class area :" << endl;
        return 0;
    }
};
class Rectangle : public Shape
{
public:
    Rectangle(int a = 0, int b = 0) : Shape(a, b) {}
    int area()
    {
        cout << "Rectangle class area :" << endl;
        return (width * height);
    }
};
class Triangle : public Shape
{
public:
    Triangle(int a = 0, int b = 0) : Shape(a, b) {}
    int area()
    {
        cout << "Triangle class area :" << endl;
        return (width * height / 2);
    }
};
// 程序的主函数
int main()
{
    Shape *shape;
    Rectangle rec(10, 7);
    Triangle tri(10, 5);

    // 存储矩形的地址
    shape = &rec;
    // 调用矩形的求面积函数 area
    shape->area();

    // 存储三角形的地址
    shape = &tri;
    // 调用三角形的求面积函数 area
    shape->area();

    return 0;
}
```

**使用virtual关键字**

在 `Shape` 类中，`area()` 的声明前放置关键字 `virtual`。

```cpp
class Shape {
   protected:
      int width, height;
   public:
      Shape( int a=0, int b=0)
      {
         width = a;
         height = b;
      }
      virtual int area()
      {
         cout << "Parent class area :" <<endl;
         return 0;
      }
};
```

结果：

```cpp
Rectangle class area
Triangle class area
```

## 虚函数

`虚函数` 是在基类中使用关键字 `virtual` 声明的函数。在派生类中重新定义基类中定义的虚函数时，会告诉编译器不要`静态链接`到该函数。

我们想要的是在程序中任意点可以根据所调用的对象类型来选择调用的函数，这种操作被称为`动态链接`，或`后期绑定`。

虚函数声明如下：`virtual ReturnType FunctionName(Parameter)`。

虚函数必须实现，如果不实现，编译器将报错，错误提示为：

```cpp
error LNK****: unresolved external symbol "public: virtual void __thiscall ClassName::virtualFunctionName(void)"
```

实例1：

`tall` 没有实现，函数表（`vtable`）的引用是未定义的，故而无法执行。但可以使用 `People people;` 然后 `people.tall();` 或 `(&people)->tall();` 因为`People`实现或者说重写、覆盖了 `Base` 的纯虚方法 `tall()`，使其在 `People` 类中有了定义，函数表挂上去了。

```cpp
#include <iostream>
using namespace std;

class Base
{
public:
    virtual void tall();
};

class People : Base
{
public:
    void tall()
    {
        cout << "people" << endl;
    };
};

int main()
{
    //Base base;//不可用

    People people; //可用
    people.tall();
    (&people)->tall();

    return 0;
}
```

实例2：

父类的虚函数或纯虚函数在子类中依然是虚函数。有时我们并不希望父类的某个函数在子类中被重写，在 C++11 及以后可以用关键字 `final` 来避免该函数再次被重写。

```cpp
#include <iostream>
using namespace std;
class Base
{
public:
    virtual void func()
    {
        cout << "This is Base" << endl;
    }
};
class _Base : public Base
{
public:
    void func() final //正确，func在Base中是虚函数
    {
        cout << "This is _Base" << endl;
    }
};
class __Base : public _Base
{
    /*    public://不正确，func在_Base中已经不再是虚函数，不能再被重写
        void func()
        {
            cout<<"This is __Base"<<endl;
        }*/
};
int main()
{
    _Base a;
    __Base b;
    Base *ptr = &a;
    ptr->func();

    ptr = &b;
    _Base *ptr2 = &b;

    ptr->func();
    ptr2->func();
}
```

结果：

```cpp
This is _Base
This is _Base
This is _Base
```

如果不希望一个类被继承，也可以使用 `final` 关键字。

格式如下：

```cpp
class Class_name final
{
};
```

C++中, `虚函数`可以为 `private`, 并且可以被子类覆盖（因为虚函数表的传递），但子类不能调用父类的`private`虚函数。虚函数的重载性和它声明的权限无关。

一个成员函数被定义为`private`属性，标志着其只能被当前类的其他成员函数(或友元函数)所访问。而`virtual`修饰符则强调父类的成员函数可以在子类中被重写，因为重写之时并没有与父类发生任何的调用关系，故而重写是被允许的。

编译器不检查虚函数的各类属性。被`virtual`修饰的成员函数，不论他们是`private、protect或是public`的，都会被统一的放置到虚函数表中。
对父类进行派生时，子类会继承到拥有相同偏移地址的虚函数表（相同偏移地址指，各虚函数相对于`VPTR`指针的偏移），则子类就会被允许对这些虚函数进行重载。
且重载时可以给重载函数定义新的属性，例如`public`，其只标志着该重载函数在该子类中的访问属性为`public`，和父类的`private`属性没有任何关系。

`纯虚函数`可以设计成私有的，不过这样不允许在本类之外的非友元函数中直接调用它，子类中只有覆盖这种纯虚函数的义务，却没有调用它的权利。

### 纯虚函数

纯虚函数声明如下：`virtual void funtion1()=0; `

纯虚函数一定没有定义，纯虚函数用来规范派生类的行为，即接口。
包含纯虚函数的类是抽象类，抽象类不能定义实例，但可以声明指向实现该抽象类的具体类的指针或引用。

虚函数可以不实现（定义）。不实现（定义）的虚函数是纯虚函数。

`= 0` 告诉编译器，函数没有主体，是纯虚函数。

```cpp
class Shape
{
protected:
    int width, height;

public:
    Shape(int a = 0, int b = 0)
    {
        width = a;
        height = b;
    }
    // pure virtual function
    virtual int area() = 0;
};
```

### 总结

* 对于虚函数来说，父类和子类都有各自的版本。由多态方式调用的时候动态绑定。
* 实现了纯虚函数的子类，该纯虚函数在子类中就编程了虚函数，子类的子类即孙子类可以覆盖该虚函数，由多态方式调用的时候动态绑定。
* 虚函数是C++中用于实现多态(`polymorphism`)的机制。核心理念就是通过基类访问派生类定义的函数。
* 在有动态分配堆上内存的时候，析构函数必须是虚函数，但没有必要是纯虚的。
* 友元不是成员函数，只有成员函数才可以是虚拟的，因此友元不能是虚拟函数。但可以通过让友元函数调用虚拟成员函数来解决友元的虚拟问题。
* 析构函数应当是虚函数，将调用相应对象类型的析构函数，因此，如果指针指向的是子类对象，将调用子类的析构函数，然后自动调用基类的析构函数。

## 动态联编的实现机制VTABLE

编译器对每个包含虚函数的类创建一个虚函数表 `VTABLE`，表中每一项指向一个虚函数的地址，即`VTABLE`表可以看成一个函数指针的数组，每个虚函数的入口地址就是这个数组的一个元素。

每个含有虚函数的类都有各自的一张虚函数表`VTABLE`。每个派生类的`VTABLE`继承了它各个基类的`VTABLE`，如果基类`VTABLE`中包含某一项（虚函数的入口地址），则其派生类的`VTABLE`中也将包含同样的一项，但是两项的值可能不同。如果派生类中重载了该项对应的虚函数，则派生类`VTABLE`的该项指向重载后的虚函数，如果派生类中没有对该项对应的虚函数进行重新定义，则使用基类的这个虚函数地址。

在创建含有虚函数的类的对象的时候，编译器会在每个对象的内存布局中增加一个`vptr`指针项，该指针指向本类的`VTABLE`。在通过指向基类对象的指针（设为`bp`）调用一个虚函数时，编译器生成的代码是先获取所指对象的`vtb1`指针，然后调用`vtb1`所指向类的`VTABLE`中的对应项（具体虚函数的入口地址）。

当基类中没有定义虚函数时，其长度=数据成员长度；
派生类长度=自身数据成员长度+基类继承的数据成员长度；

当基类中定义虚函数后，其长度=数据成员长度+虚函数表的地址长度；
派生类长度=自身数据成员长度+基类继承的数据成员长度+虚函数表的地址长度。

包含一个虚函数和几个虚函数的类的长度增量为0。含有虚函数的类只是增加了一个指针用于存储虚函数表的首地址。

派生类与基类同名的虚函数在`VTABLE`中有相同的索引号（或序号）。

参考：

[C++ 多态](https://www.runoob.com/cplusplus/cpp-polymorphism.html)