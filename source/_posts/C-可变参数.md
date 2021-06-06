---
title: C-可变参数
date: 2021-06-06 14:54:05
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

C 语言允许定义一个函数，能根据具体的需求接受可变数量的参数。

第一个参数都是表示被传的可变参数的总数，省略号被用来传递可变数量的参数。

```c
int func(int, ... ) 
{
   .
   .
   .
}
 
int main()
{
   func(2, 2, 3);
   func(3, 2, 3, 4);
}
```

函数 `func()` 最后一个参数写成省略号，即三个点号（`...`），省略号之前的那个参数是 `int`，代表了要传递的可变参数的总数。

<!--more-->
为了使用这个功能，需要使用 `stdarg.h` 头文件，该文件提供了实现可变参数功能的函数和宏。

具体步骤如下：

* 定义一个函数，最后一个参数为省略号，省略号前面可以设置自定义参数。
* 在函数定义中创建一个 `va_list` 类型变量，该类型是在 `stdarg.h` 头文件中定义的。
* 使用 `int` 参数和 `va_start` 宏来初始化 `va_list` 变量为一个参数列表。宏 `va_start` 是在 `stdarg.h` 头文件中定义的。
* 使用 `va_arg` 宏和 `va_list` 变量来访问参数列表中的每个项。
* 使用宏 `va_end` 来清理赋予 `va_list` 变量的内存。

C 函数要在程序中用到以下这些宏:

```c
void va_start( va_list arg_ptr, prev_param ); 
type va_arg( va_list arg_ptr, type ); 
void va_end( va_list arg_ptr );
```

`va_list`: 用来保存宏 `va_start`、`va_arg` 和 `va_end` 所需信息的一种类型。为了访问变长参数列表中的参数，必须声明 `va_list` 类型的一个对象，定义： `typedef char * va_list;`

`va_start`: 访问变长参数列表中的参数之前使用的宏，它初始化用 `va_list` 声明的对象，初始化结果供宏 `va_arg` 和 `va_end` 使用；

`va_arg`: 展开成一个表达式的宏，该表达式具有变长参数列表中下一个参数的值和类型。每次调用 `va_arg` 都会修改用 `va_list` 声明的对象，从而使该对象指向参数列表中的下一个参数；

`va_end`: 该宏使程序能够从变长参数列表用宏 `va_start` 引用的函数中正常返回。

`va` 在这里是 `variable-argument(可变参数)` 的意思。

这些宏定义在 `stdarg.h` 中，所以用到可变参数的程序应该包含这个头文件。

实例1:

一个带有可变数量参数的函数，并返回它们的平均值：

```c
#include <stdio.h>
#include <stdarg.h>
 
double average(int num,...)
{
    va_list valist;
    double sum = 0.0;
    int i;
 
    /* 为 num 个参数初始化 valist */
    va_start(valist, num);
 
    /* 访问所有赋给 valist 的参数 */
    for (i = 0; i < num; i++)
    {
       sum += va_arg(valist, int);
    }
    /* 清理为 valist 保留的内存 */
    va_end(valist);
 
    return sum/num;
}
 
int main()
{
   printf("Average of 2, 3, 4, 5 = %f\n", average(4, 2,3,4,5));
   printf("Average of 5, 10, 15 = %f\n", average(3, 5,10,15));
}
```

结果：

```c
Average of 2, 3, 4, 5 = 3.500000
Average of 5, 10, 15 = 10.000000
```

实例2：

```c
#include <stdio.h>

void debug_arg(unsigned int num, ...) 
{
    unsigned int i = 0;
    unsigned int *addr = &num;
    for (i = 0; i <= num; i++) 
    {
        /* *(addr + i) 从左往右依次取出传递进来的参数,类似于出栈过程 */
        printf("i=%d,value=%d\r\n", i, *(addr + i));
    }
}
int main(void)
{
    debug_arg(3, 66, 88, 666);
    return 0;
}
```

结果：

```c
i=0,value=3
i=1,value=0
i=2,value=66
i=3,value=0
```

可变参数的工作原理,以32位机为例:

* 1.函数参数的传递存储在栈中,从右至左压入栈中,压栈过程为递减。
* 2.参数的传递以4字节对齐

实例3：

```c
#include <stdio.h>
#include <string.h>
#include <stdarg.h>

/* ANSI标准形式的声明方式，括号内的省略号表示可选参数 */  
int demo(char *msg, ... )  
{  
    va_list argp;                    /* 定义保存函数参数的结构 */  
    int argno = 0;                    /* 纪录参数个数 */  
    char *para;                        /* 存放取出的字符串参数 */                                      
    va_start( argp, msg );          /* argp指向传入的第一个可选参数，      msg是最后一个确定的参数 */  
    
    while (1) 
    {  
        para = va_arg( argp, char *);                 /*      取出当前的参数，类型为char *. */  
        if ( strcmp( para, "/0") == 0 )  
                                                      /* 采用空串指示参数输入结束 */  
            break;  
        printf("Parameter #%d is: %s/n", argno, para);  
        argno++;  
    }  
    va_end( argp );                                   /* 将argp置为NULL */  
    return 0;  
}

int main( void )  
{  
    demo("DEMO", "This", "is", "a", "demo!" ,"333333", "/0");  
} 
```

[C 可变参数](https://www.runoob.com/cprogramming/c-variable-arguments.html)