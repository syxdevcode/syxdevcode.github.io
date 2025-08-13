---
title: C-输入和输出
date: 2021-06-04 22:15:11
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

## 标准文件

C 语言把所有的设备都当作文件。所以设备（比如显示器）被处理的方式与文件相同。以下三个文件会在程序执行时自动打开，以便访问键盘和屏幕。

| 标准文件	| 文件指针	| 设备 |
|:----  |:---- | :---- |
| 标准输入| 	stdin  | 	键盘| 
| 标准输出| 	stdout | 	屏幕| 
| 标准错误| 	stderr | 	屏幕| 

<!--more-->
## scanf() 和 printf() 函数

* `int scanf(const char *format, ...)` 函数从标准输入流 `stdin` 读取输入，并根据提供的 format 来浏览输入。
* `int printf(const char *format, ...)` 函数把输出写入到标准输出流 `stdout` ，并根据提供的格式产生输出。

`format` 可以是一个简单的常量字符串，但是可以分别指定 `%s、%d、%c、%f` 等来输出或读取字符串、整数、字符或浮点数。

```c
#include <stdio.h>

int main( ) {
 
   char str[100];
   int i;
 
   printf( "Enter a value :");
   scanf("%s %d", str, &i);
 
   printf( "\nYou entered: %s %d ", str, i);
   printf("\n");
   return 0;
}
```

`scanf()` 期待输入的格式与给出的 `%s` 和 `%d` 相同，这意味着必须提供有效的输入，比如 `string integer`，如果提供的是 `string string` 或 `integer integer`，它会被认为是错误的输入。另外，在读取字符串时，只要遇到一个空格，`scanf()` 就会停止读取，所以 `this is test` 对 `scanf()` 来说是三个字符串。

## getchar() & putchar() 函数

* `int getchar(void)` 函数从屏幕读取下一个可用的字符，并把它返回为一个整数。这个函数在同一个时间内只会读取一个单一的字符。可以在循环内使用这个方法，以便从屏幕上读取多个字符。
* `int putchar(int c)` 函数把字符输出到屏幕上，并返回相同的字符。这个函数在同一个时间内只会输出一个单一的字符。可以在循环内使用这个方法，以便在屏幕上输出多个字符。

实例

```c
#include <stdio.h>
 
int main( )
{
   int c;
 
   printf( "Enter a value :");
   c = getchar( );
 
   printf( "\nYou entered: ");
   putchar( c );
   printf( "\n");
   return 0;
}
```

结果：

```c
$./a.out
Enter a value :runoob

You entered: r
```

## gets() & puts() 函数

* char *gets(char *s) 函数从 stdin 读取一行到 s 所指向的缓冲区，直到一个终止符或 EOF。
* int puts(const char *s) 函数把字符串 s 和一个尾随的换行符写入到 stdout。

实例

```c
#include <stdio.h>
 
int main( )
{
   char str[100];
 
   printf( "Enter a value :");
   gets( str );
 
   printf( "\nYou entered: ");
   puts( str );
   return 0;
}
```

结果：

```c
$./a.out
Enter a value :runoob

You entered: runoob
```

linux系统下需要这样编译：不支持 `gets` 与 `puts`, 需要用 `fgets` 和 `fputs`。

将以下代码放到 test.c 文件：

```c
#include <stdio.h>

int main()
{
    char c[100];
    printf("Enter a value:");
    fgets( c,100,stdin );

    printf("\nyou entered:");
    fputs( c,stdout );

    return 0;
}
```

编译执行以上代码，输出结果为：

```c
# gcc test.c 
# ./a.out 
Enter a value:runoob

you entered:runoob
```

[C 输入 & 输出](https://www.runoob.com/cprogramming/c-input-output.html)