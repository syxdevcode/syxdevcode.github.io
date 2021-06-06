---
title: C-typedef
date: 2021-06-04 09:01:01
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

C 语言提供了 `typedef` 关键字，可以使用它来为类型取一个新的名字。下面的实例为单字节数字定义了一个术语 `BYTE`：

```c
typedef unsigned char BYTE;
```

在这个类型定义之后，标识符 `BYTE` 可作为类型 `unsigned char` 的缩写，例如：

```c
BYTE  b1, b2;
```

按照惯例，定义时会大写字母，以便提醒用户类型名称是一个象征性的缩写，但您也可以使用小写字母，如下：

```c
typedef unsigned char byte;
```

<!--more-->
可以使用 `typedef` 来为用户自定义的数据类型取一个新的名字。例如，可以对结构体使用 `typedef` 来定义一个新的数据类型名字，然后使用这个新的数据类型来直接定义结构变量，如下：

实例

```c
#include <stdio.h>
#include <string.h>
 
typedef struct Books
{
   char  title[50];
   char  author[50];
   char  subject[100];
   int   book_id;
} Book;
 
int main( )
{
   Book book;
 
   strcpy( book.title, "C 教程");
   strcpy( book.author, "Runoob"); 
   strcpy( book.subject, "编程语言");
   book.book_id = 12345;
 
   printf( "书标题 : %s\n", book.title);
   printf( "书作者 : %s\n", book.author);
   printf( "书类目 : %s\n", book.subject);
   printf( "书 ID : %d\n", book.book_id);
 
   return 0;
}
```

结果：

```c
书标题 : C 教程
书作者 : Runoob
书类目 : 编程语言
书 ID : 12345
```

## typedef vs #define

`#define` 是 C 指令，用于为各种数据类型定义别名，与 `typedef` 类似，但是它们有以下几点不同：

* `typedef` 仅限于为类型定义符号名称，`#define` 不仅可以为类型定义别名，也能为数值定义别名，比如可以定义 1 为 ONE。
* `typedef` 是由编译器执行解释的，`#define` 语句是由预编译器进行处理的。

实例

```c
#include <stdio.h>
 
#define TRUE  1
#define FALSE 0
 
int main( )
{
   printf( "TRUE 的值: %d\n", TRUE);
   printf( "FALSE 的值: %d\n", FALSE);
 
   return 0;
}
```

结果：

```
TRUE 的值: 1
FALSE 的值: 0
```

（1）`#define`可以使用其他类型说明符对宏类型名进行扩展，但对 `typedef` 所定义的类型名却不能这样做。例如：

```c
#define INTERGE int;
unsigned INTERGE n;  //没问题
typedef int INTERGE;
unsigned INTERGE n;  //错误，不能在 INTERGE 前面添加 unsigned
```

（2） 在连续定义几个变量的时候，`typedef` 能够保证定义的所有变量均为同一类型，而 `#define` 则无法保证。例如：

```c
#define PTR_INT int *
PTR_INT p1, p2;        //p1、p2 类型不相同，宏展开后变为int *p1, p2;
typedef int * PTR_INT
PTR_INT p1, p2;        //p1、p2 类型相同，它们都是指向 int 类型的指针。
```

用 `typedef` 为数组去别名：

```c
typedef int A[6];
```

表示用 A 代替 `int [6]`。
即：A a; 等于 `int a[6]`;

`typedef` 还有一个作用，就是为复杂的声明定义一个新的简单的别名。用在回调函数中特别好用：

1. 原声明：`int *(*a[5])(int, char*);`

在这里，变量名为 a，直接用一个新别名 `pFun` 替换 a 就可以了：

```c
typedef int *(*pFun)(int, char*);
```

于是，原声明的最简化版：

```c
pFun a[5];
```

由 `typedef` 定义的函数 `pFun`，为一个新的类型，所以这个新的类型可以像 int 一样定义变量，于是，`pFun a[5]`; 就定义了 `int *(*a[5])(int, char*);`

2. 原声明：`void (*b[10]) (void (*)());`

这里，变量名为 b，先替换右边部分括号里的，`pFunParam` 为别名一：

```c
typedef void (*pFunParam)();
```

再替换左边的变量 b，`pFunx` 为别名二：

```c
typedef void (*pFunx)(pFunParam);
```

于是，原声明的最简化版：

```c
pFunx b[10];
```

另外，也要注意，`typedef` 在语法上是一个存储类的关键字（如 `auto`、`extern`、`mutable`、`static`、`register` 等一样），虽然它并不真正影响对象的存储特性，如：

```c
typedef static int INT2; // 不可行
// 编译将失败，会提示“指定了一个以上的存储类”。
```

[C typedef](https://www.runoob.com/cprogramming/c-typedef.html)