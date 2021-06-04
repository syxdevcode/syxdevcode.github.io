---
title: C-共用体
date: 2021-06-03 22:22:48
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

共用体是一种特殊的数据类型，允许您在相同的内存位置存储不同的数据类型。您可以定义一个带有多成员的共用体，但是任何时候只能有一个成员带有值。共用体提供了一种使用相同的内存位置的有效方式。

## 定义共用体

为了定义共用体，您必须使用 `union` 语句，方式与定义结构类似。`union` 语句定义了一个新的数据类型，带有多个成员。`union` 语句的格式如下：

```c
union [union tag]
{
   member definition;
   member definition;
   ...
   member definition;
} [one or more union variables];
```

`union tag` 是可选的，每个 `member definition` 是标准的变量定义，比如 `int i;` 或者 `float f;` 或者其他有效的变量定义。在共用体定义的末尾，最后一个分号之前，您可以指定一个或多个共用体变量，这是可选的。下面定义一个名为 `Data` 的共用体类型，有三个成员 `i、f 和 str`：

```c
union Data
{
   int i;
   float f;
   char  str[20];
} data;
```

<!--more-->
现在，`Data` 类型的变量可以存储一个整数、一个浮点数，或者一个字符串。这意味着一个变量（相同的内存位置）可以存储多个多种类型的数据。您可以根据需要在一个共用体内使用任何内置的或者用户自定义的数据类型。

共用体占用的内存应足够存储共用体中最大的成员。例如，在上面的实例中，`Data` 将占用 `20` 个字节的内存空间，因为在各个成员中，字符串所占用的空间是最大的。下面的实例将显示上面的共用体占用的总内存大小：

实例

```c
#include <stdio.h>
#include <string.h>
 
union Data
{
   int i;
   float f;
   char  str[20];
};
 
int main( )
{
   union Data data;        
 
   printf( "Memory size occupied by data : %d\n", sizeof(data));
 
   return 0;
}
```

结果:

```c
Memory size occupied by data : 20
```

## 访问共用体成员

为了访问共用体的成员，我们使用`成员访问运算符（.）`。成员访问运算符是共用体变量名称和我们要访问的共用体成员之间的一个句号。您可以使用 `union` 关键字来定义共用体类型的变量。下面的实例演示了共用体的用法：

实例

```c
#include <stdio.h>
#include <string.h>
 
union Data
{
   int i;
   float f;
   char  str[20];
};
 
int main( )
{
   union Data data;        
 
   data.i = 10;
   data.f = 220.5;
   strcpy( data.str, "C Programming");
 
   printf( "data.i : %d\n", data.i);
   printf( "data.f : %f\n", data.f);
   printf( "data.str : %s\n", data.str);
 
   return 0;
}
```

结果：

```c
data.i : 1917853763
data.f : 4122360580327794860452759994368.000000
data.str : C Programming
```

在这里，可以看到共用体的 `i 和 f` 成员的值有损坏，因为最后赋给变量的值占用了内存位置，这也是 `str` 成员能够完好输出的原因。

同一时间只使用一个变量：

实例

```c
#include <stdio.h>
#include <string.h>
 
union Data
{
   int i;
   float f;
   char  str[20];
};
 
int main( )
{
   union Data data;        
 
   data.i = 10;
   printf( "data.i : %d\n", data.i);
   
   data.f = 220.5;
   printf( "data.f : %f\n", data.f);
   
   strcpy( data.str, "C Programming");
   printf( "data.str : %s\n", data.str);
 
   return 0;
}
```

结果：

```c
data.i : 10
data.f : 220.500000
data.str : C Programming
```

## 共用体字节对齐

```c
union Data{
    int i;
    float f;
    char str[9];
    double d;
}data;
```

对齐原则：

* 【原则1】数据成员对齐规则：结构(struct)(或联合(union))的数据成员，第一个数据成员放在offset为0的地方，以后每个数据成员的对齐按照 `#pragma pack` 指定的数值和这个数据成员自身长度中，比较小的那个进行。
* 【原则2】结构(或联合)的整体对齐规则：在数据成员完成各自对齐之后，结构(或联合)本身也要进行对齐，对齐将按照 `#pragma pack` 指定的数值和结构(或联合)最大数据成员长度中，比较小的那个进行。
* 【原则3】结构体作为成员：如果一个结构里有某些结构体成员，则结构体成员要从其内部最大元素大小的整数倍地址开始存储。

按照原则2，VS 中默认的是 `#pragma park(8)`，`char[9]` 长度为 9。

```c
8<9;
```

按照 8 的倍数且 `>9`，则取为 `16`；

注：`#pragma pack`：程序编译器对结构的存储的特殊处理确实提高CPU存储变量的速度，但是有时候也带来了一些麻烦，我们也屏蔽掉变量默认的对齐方式，自己可以设定变量的对齐方式。

## 共用体应用场景

通信中的数据包会用到共用体:因为不知道对方会发一个什么包过来，用共用体的话就很简单了，定义几种格式的包，收到包之后就可以直接根据包的格式取出数据。

[C 共用体](https://www.runoob.com/cprogramming/c-unions.html)