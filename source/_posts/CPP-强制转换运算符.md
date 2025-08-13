---
title: CPP-强制转换运算符
date: 2021-06-28 17:25:33
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

强制转换运算符是一种特殊的运算符，它把一种数据类型转换为另一种数据类型。强制转换运算符是一元运算符，它的优先级与其他一元运算符相同。

大多数的 C++ 编译器都支持大部分通用的强制转换运算符：

```cpp
(type) expression 
```

其中，type 是转换后的数据类型。

## C++强制转换运算符

c++ 有四种显示类型转换，分别为 `static_cast`, `dynamic_cast`, `const_cast`, `reinterpret_cast`。

<!--more-->
### static_cast<type> (expr)

`static_cast`:强制将一种数据类型转换为另一种数据类型。`static_cast` 可以实现C++中内置基本数据类型之间的相互转换。例如，将整型数据转换为浮点型数据。

如果涉及到类的话，`static_cast` 只能在有相互联系的类型中进行相互转换,不一定包含虚函数。

```cpp
int a = 10;
int b = 3;
double result = static_cast<double>(a) / static_cast<double>(b);
```

`static_cast` 是 `静态转换` 的意思，也就是在编译期间转换，转换失败的话会抛出一个编译错误，没有运行时类检查来保证转换的安全性。例如，它可以用来把一个基类指针转换为派生类指针。

`static_cast` 只能用于良性转换，如：

* 原有的自动类型转换，例如 `short` 转 `int`、`int` 转 `double`、`const` 转非 `const`、向上转型等；
* `void` 指针和具体类型指针之间的转换，例如 `void *` 转 `int *`、`char *` 转 `void *` 等；
* 有转换构造函数或者类型转换函数的类与其它类型之间的转换，例如 `double` 转 `Complex`（调用转换构造函数）、`Complex` 转 `double`（调用类型转换函数）。

它主要有几种用法：
* （1）用于类层次结构中基类和派生类之间指针或引用的转换；
      进行上行转换（把派生类的指针或引用转换成基类表示）是安全的；
      进行下行转换（把基类的指针或引用转换为派生类表示），由于没有动态类型检查，所以是不安全的。
* （2）用于基本数据类型之间的转换，如把 `int` 转换成 `char`。这种转换的安全也要开发人员来保证
* （3）把空指针转换成目标类型的空指针
* （4）把任何类型的表达式转换为`void`类型

注意：`static_cast` 不能转换掉`expression`的`const`、`volitale`或者`__unaligned`属性。

### dynamic_cast<type> (expr)

`dynamic_cast` 在运行时执行转换，运行时要进行类型检查，验证转换的有效性。如果转换未执行，则转换失败，表达式 `expr` 被判定为 null。`dynamic_cast` 执行动态转换时，type 必须是类的指针、类的引用或者 `void*`，如果 type 是类指针类型，那么 expr 也必须是一个指针，如果 type 是一个引用，那么 expr 也必须是一个引用。

1，不能用于内置的基本数据类型的强制转换。
2，`dynamic_cast` 转换如果成功的话返回的是指向类的指针或引用，转换失败的话则会返回NULL。
3，使用 `dynamic_cast` 进行转换的，基类中一定要有虚函数，否则编译不通过。
4，在类的转换时，在类层次间进行上行转换时，`dynamic_cast`和`static_cast`的效果是一样的。在进行下行转换时，`dynamic_cast`具有类型检查的功能，比 `static_cast`更安全。

向上转换，即为子类指针指向父类指针（一般不会出问题）；向下转换，即将父类指针转化子类指针。
向下转换的成功与否还与将要转换的类型有关，即要转换的指针指向的对象的实际类型与转换以后的对象类型一定要相同，否则转换失败。

在C++中，编译期的类型转换有可能会在运行时出现错误，特别是涉及到类对象的指针或引用操作时，更容易产生错误。`Dynamic_cast` 操作符则可以在运行期对可能产生问题的类型转换进行测试。

实例：

```cpp
#include<iostream>
using  namespace  std;
 
class  base
{
public  :
     void  m(){cout<< "m" <<endl;}
};
 
class  derived :  public  base
{
public :
     void  f(){cout<< "f" <<endl;}
};
 
int  main()
{
     derived * p;
     p =  new  base;
     p =  static_cast <derived *>( new  base);
     p->m();
     p->f();
     return  0;
}
```

### const_cast<type> (expr)

`const_cast` 主要是用于改变类型的常量属性：
常量指针转化为非常量指针；
常量引用转化为非常量引用；
常量对象转化为非常量对象。

`const_cast` 用于强制去掉这种不能被修改的常数特性，但需要特别注意的是 `const_cast` 不是用于去除变量的常量性，而是去除指向常数对象的指针或引用的常量性，其去除常量性的对象必须为指针或引用。

用法：`const_cast<type_id> (expression)`

1，该运算符用来修改类型的`const`或`volatile`属性。除了`const` 或`volatile`修饰之外， `type_id` 和`expression` 的类型是一样的。
2，常量指针被转化成非常量指针，并且仍然指向原来的对象；
3，常量引用被转换成非常量引用，并且仍然指向原来的对象；
4，常量对象被转换成非常量对象。

```cpp
#include<iostream>
using  namespace  std;
 
int  main()
{
     const  int  a = 10;
     const  int  *p = &a;
     int  *q;
     q =  const_cast<int *>(p);
     *q = 20;     //fine
     cout <<a<< " " <<*p<< " " <<*q<<endl;
         cout <<&a<< " " <<p<< " " <<q<<endl;
     return  0;
}
```

### reinterpret_cast<type> (expr)

`reinterpret_cast` 运算符把某种指针改为其他类型的指针。`reinterpret_cast` 用于非关联类型的转换，操作结果是一个指针到其他指针的二进制拷贝，没有类型检查。

`reinterpret_cast` 主要有三种强制转换用途：改变指针或引用的类型、将指针或引用转换为一个足够长度的整形、将整型转换为指针或引用类型。

用法：reinterpret_cast<type_id> (expression)
    
`type-id` 必须是一个指针、引用、算术类型、函数指针或者成员指针。
它可以把一个指针转换成一个整数，也可以把一个整数转换成一个指针（先把一个指针转换成一个整数，在把该整数转换成原类型的指针，还可以得到原先的指针值）。
在使用 `reinterpret_cast` 强制转换过程仅仅只是比特位的拷贝，因此在使用过程中需要特别谨慎！

```cpp
int *a = new int;
double *d = reinterpret_cast<double *>(a);
```

[C++ 强制转换运算符](https://www.runoob.com/cplusplus/cpp-casting-operators.html)

[cpp四种类型转换](https://blog.csdn.net/gettogetto/article/details/76773150)