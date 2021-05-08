---
title: C变量
date: 2021-05-07 16:32:54
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

C 中每个变量都有特定的类型，类型决定了变量存储的大小和布局，该范围内的值都可以存储在内存中，运算符可应用于变量上。

## C中的变量定义

变量定义就是告诉编译器在何处创建变量的存储，以及如何创建变量的存储。变量定义指定一个数据类型，并包含了该类型的一个或多个变量的列表，格式：

```c
type variable_list;
```

type 必须是一个有效的 C 数据类型，可以是 `char、w_char、int、float、double` 或任何用户自定义的对象，`variable_list` 可以由一个或多个标识符名称组成，多个标识符之间用逗号分隔。

```c
int    i, j, k;
char   c, ch;
float  f, salary;
double d;
```

初始化器由一个等号，后跟一个常量表达式组成，如下所示：

```c
type variable_name = value;
```

下面列举几个实例：

```c
extern int d = 4, f = 5;    // d 和 f 的声明与初始化
int d = 3, f = 5;           // 定义并初始化 d 和 f
char x = 'x';               // 变量 x 的值为 'x'
```

不带初始化的定义：带有静态存储持续时间的变量会被隐式初始化为 NULL（所有字节的值都是 0），其他所有变量的初始值是未定义的。

## C中的变量声明

变量声明向编译器保证变量以指定的类型和名称存在，这样编译器在不需要知道变量完整细节的情况下也能继续进一步的编译。变量声明只在编译时有它的意义，在程序连接时编译器需要实际的变量声明。

变量的声明有两种情况：

* 1、一种是需要建立存储空间的。例如：`int a` 在声明的时候就已经建立了存储空间。
* 2、另一种是不需要建立存储空间的，通过使用 `extern` 关键字声明变量名而不定义它。 例如：`extern int a` 其中变量 a 可以在别的文件中定义的。
* 3，除非有 `extern` 关键字，否则都是变量的定义。

```c
extern int i; //声明，不是定义
int i; //声明，也是定义
```

**实例**

变量在头部就已经被声明，但是定义与初始化在主函数内：

```c
#include <stdio.h>
 
// 函数外定义变量 x 和 y
int x;
int y;
int addtwonum()
{
    // 函数内声明变量 x 和 y 为外部变量
    extern int x;
    extern int y;
    // 给外部变量（全局变量）x 和 y 赋值
    x = 1;
    y = 2;
    return x+y;
}
 
int main()
{
    int result;
    // 调用函数 addtwonum
    result = addtwonum();
    
    printf("result 为: %d",result);
    return 0;
}
```

结果：

```c
result 为: 3
```

###  extern 关键字

如果需要在一个源文件中引用另外一个源文件中定义的变量，我们只需在引用的文件中将变量加上 extern 关键字的声明即可。

`addtwonum.c` 文件代码：

```c
#include <stdio.h>
/*外部变量声明*/
extern int x ;
extern int y ;
int addtwonum()
{
    return x+y;
}
```

`test.c` 文件代码：
注意：vs code需要编码格式为 gb2312。

```c
#include <stdio.h>
  
/*定义两个全局变量*/
int x=1;
int y=2;
int addtwonum();
int main(void)
{
    int result;
    result = addtwonum();
    printf("result 为: %d\n",result);
    return 0;
}
```

当上面的代码被编译和执行时，它会产生下列结果：

```sh
$ gcc addtwonum.c test.c -o main
$ ./main
result 为: 3
```

## C 中的左值（Lvalues）和右值（Rvalues）

C 中有两种类型的表达式：

* 左值（lvalue）：指向内存位置的表达式被称为左值（lvalue）表达式。左值可以出现在赋值号的左边或右边。
* 右值（rvalue）：术语右值（rvalue）指的是存储在内存中某些地址的数值。右值是不能对其进行赋值的表达式，也就是说，右值可以出现在赋值号的右边，但不能出现在赋值号的左边。

变量是左值，因此可以出现在赋值号的左边。数值型的字面值是右值，因此不能被赋值，不能出现在赋值号的左边。

下面是一个有效的语句：

```c
int g = 20;
```

但是下面这个就不是一个有效的语句，会生成编译时错误：

```c
10 = 20;
```

## 全局变量和局部变量在内存中的区别

全局变量保存在内存的全局存储区中，占用静态的存储单元；局部变量保存在栈中，只有在所在函数被调用时才动态地为变量分配存储单元。

C语言经过编译之后将内存分为以下几个区域：

（1）栈（stack）：由编译器进行管理，自动分配和释放，存放函数调用过程中的各种参数、局部变量、返回值以及函数返回地址。操作方式类似数据结构中的栈。
（2）堆（heap）：用于程序动态申请分配和释放空间。C语言中的malloc和free，C++中的new和delete均是在堆中进行的。正常情况下，程序员申请的空间在使用结束后应该释放，若程序员没有释放空间，则程序结束时系统自动回收。注意：这里的“堆”并不是数据结构中的“堆”。（3）全局（静态）存储区：分为DATA段和BSS段。DATA段（全局初始化区）存放初始化的全局变量和静态变量；BSS段（全局未初始化区）存放未初始化的全局变量和静态变量。程序运行结束时自动释放。其中BBS段在程序执行之前会被系统自动清0，所以未初始化的全局变量和静态变量在程序执行之前已经为0。
（4）文字常量区：存放常量字符串。程序结束后由系统释放。
（5）程序代码区：存放程序的二进制代码。

C语言中的全局变量包括外部变量和静态变量，均是保存在全局存储区中，占用永久性的存储单元；局部变量，即自动变量，保存在栈中，只有在所在函数被调用时才由系统动态在栈中分配临时性的存储单元。

```c
#include <stdio.h>
#include <stdlib.h>
int k1 = 1;
int k2;
static int k3 = 2;
static int k4;
int main( )
{  
    static int m1=2, m2;
    int i=1;
    char*p;
    char str[10] = "hello";
    char*q = "hello";
    p= (char *)malloc( 100 );
    free(p);
    printf("栈区-变量地址  i：%p\n", &i);
    printf("                p：%p\n", &p);
    printf("              str：%p\n", str);
    printf("                q：%p\n", &q);
    printf("堆区地址-动态申请：%p\n", p);
    printf("全局外部有初值 k1：%p\n", &k1);
    printf("    外部无初值 k2：%p\n", &k2);
    printf("静态外部有初值 k3：%p\n", &k3);
    printf("    外静无初值 k4：%p\n", &k4);
    printf("  内静态有初值 m1：%p\n", &m1);
    printf("  内静态无初值 m2：%p\n", &m2);
    printf("文字常量地址    ：%p, %s\n",q, q);
    printf("程序区地址      ：%p\n",&main);
    return 0;
}
```