---
title: C-错误处理
date: 2021-06-06 14:37:52
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

C 语言不提供对错误处理的直接支持，但是作为一种系统编程语言，它以返回值的形式允许您访问底层数据。在发生错误时，大多数的 C 或 `UNIX` 函数调用返回 1 或 `NULL`，同时会设置一个错误代码 `errno`，该错误代码是全局变量，表示在函数调用期间发生了错误。

可以在 `errno.h` 头文件中找到各种各样的错误代码。

所以，C 程序员可以通过检查返回值，然后根据返回值决定采取哪种适当的动作。

在程序初始化时，把 `errno` 设置为 0，0 值表示程序中没有错误。

<!--more-->
## errno、perror() 和 strerror()

C 语言提供了 `perror()` 和 `strerror()` 函数来显示与 `errno` 相关的文本消息。

`perror()` 函数显示传给它的字符串，后跟一个冒号、一个空格和当前 `errno` 值的文本表示形式。
`strerror()` 函数，返回一个指针，指针指向当前 `errno` 值的文本表示形式。

模拟一种错误情况，尝试打开一个不存在的文件。

可以使用多种方式来输出错误消息，在这里我们使用函数来演示用法。另外有一点需要注意，应该使用 `stderr` 文件流来输出所有的错误。

实例

```c
#include <stdio.h>
#include <errno.h>
#include <string.h>
 
extern int errno ;
 
int main ()
{
   FILE * pf;
   int errnum;
   pf = fopen ("unexist.txt", "rb");
   if (pf == NULL)
   {
      errnum = errno;
      fprintf(stderr, "错误号: %d\n", errno);
      perror("通过 perror 输出错误");
      fprintf(stderr, "打开文件错误: %s\n", strerror( errnum ));
   }
   else
   {
      fclose (pf);
   }
   return 0;
}
```

结果：

```c
错误号: 2
通过 perror 输出错误: No such file or directory
打开文件错误: No such file or directory
```

## 被零除的错误

在进行除法运算时，如果不检查除数是否为零，则会导致一个运行时错误。

为了避免这种情况发生，下面的代码在进行除法运算前会先检查除数是否为零：

实例

```c
#include <stdio.h>
#include <stdlib.h>
 
int main()
{
   int dividend = 20;
   int divisor = 0;
   int quotient;
 
   if( divisor == 0){
      fprintf(stderr, "除数为 0 退出运行...\n");
      exit(-1);
   }
   quotient = dividend / divisor;
   fprintf(stderr, "quotient 变量的值为 : %d\n", quotient );
 
   exit(0);

   return 0;
}
```

结果：

```
除数为 0 退出运行...
```

## 程序退出状态

通常情况下，程序成功执行完一个操作正常退出的时候会带有值 `EXIT_SUCCESS`。在这里，`EXIT_SUCCESS` 是宏，它被定义为 0。

如果程序中存在一种错误情况，当退出程序时，会带有状态值 `EXIT_FAILURE`，被定义为 `-1`。所以，上面的程序可以写成：

实例

```c
#include <stdio.h>
#include <stdlib.h>
 
int main()
{
   int dividend = 20;
   int divisor = 5;
   int quotient;
 
   if( divisor == 0){
      fprintf(stderr, "除数为 0 退出运行...\n");
      exit(EXIT_FAILURE);
   }
   quotient = dividend / divisor;
   fprintf(stderr, "quotient 变量的值为: %d\n", quotient );
 
   exit(EXIT_SUCCESS);

   return 0;
}
```

结果：

```
quotient 变量的值为 : 4
```

[C 错误处理](https://www.runoob.com/cprogramming/c-error-handling.html)