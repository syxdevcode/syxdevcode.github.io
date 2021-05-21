---
title: C-循环语法
date: 2021-05-20 09:27:27
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

* while 循环
* for 循环
* do...while 循环

**break 语句**	终止循环或 switch 语句，程序流将继续执行紧接着循环或 switch 的下一条语句。
**continue 语句**	告诉一个循环体立刻停止本次循环迭代，重新开始下次循环迭代。

## while循环

```c
#include <stdio.h>
 
int main ()
{
   /* 局部变量定义 */
   int a = 10;

   /* while 循环执行 */
   while( a < 20 )
   {
      printf("a 的值： %d\n", a);
      a++;
   }
 
   return 0;
}
```

## for循环

```c
#include <stdio.h>
 
int main ()
{
   /* for 循环执行 */
   for( int a = 10; a < 20; a = a + 1 )
   {
      printf("a 的值： %d\n", a);
   }
 
   return 0;
}
```

## do...while循环

```c
#include <stdio.h>
 
int main ()
{
   /* 局部变量定义 */
   int a = 10;

   /* do 循环执行，在条件被测试之前至少执行一次 */
   do
   {
       printf("a 的值： %d\n", a);
       a = a + 1;
   }while( a < 20 );
 
   return 0;
}
```

##  goto语句

注意：在任何编程语言中，都不建议使用 `goto` 语句。因为它使得程序的控制流难以跟踪，使程序难以理解和难以修改。任何使用 `goto` 语句的程序可以改写成不需要使用 `goto` 语句的写法。

`goto` 语句的语法：

```c
goto label;
..
.
label: statement;
```

label 可以是任何除 C 关键字以外的纯文本，它可以设置在 C 程序中 `goto` 语句的前面或者后面。

![goto.png](/img/goto.png)

```c
#include <stdio.h>
 
int main ()
{
   /* 局部变量定义 */
   int a = 10;

   /* do 循环执行 */
   LOOP:do
   {
      if( a == 15)
      {
         /* 跳过迭代 */
         a = a + 1;
         goto LOOP;
      }
      printf("a 的值： %d\n", a);
      a++;
     
   }while( a < 20 );
 
   return 0;
}
```

## 无限循环

`for` 循环在传统意义上可用于实现无限循环。由于构成循环的三个表达式中任何一个都不是必需的，您可以将某些条件表达式留空来构成一个无限循环。

实例

```c
#include <stdio.h>
 
int main ()
{
   for( ; ; )
   {
      printf("该循环会永远执行下去！\n");
   }
   return 0;
}
```

当条件表达式不存在时，它被假设为真。您也可以设置一个初始值和增量表达式，但是一般情况下，C 程序员偏向于使用 `for(;;)` 结构来表示一个无限循环。

注意：您可以按 `Ctrl + C` 键终止一个无限循环。





























