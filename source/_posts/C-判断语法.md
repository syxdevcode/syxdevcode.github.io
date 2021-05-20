---
title: C-判断语法
date: 2021-05-20 09:08:29
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

C 语言把任何**非零**和**非空**的值假定为 `true`，把**零**或 **null** 假定为 `false`。

## if 语句

```c
#include <stdio.h>
 
int main ()
{
   int a = 10;
 
   if( a < 20 )
   {
       printf("a 小于 20\n" );
   }
   return 0;
}
```

<!--more-->
## if...else 语句

```c
#include <stdio.h>
 
int main ()
{
   int a = 100;
 
   if( a < 20 )
   {
       printf("a 小于 20\n" );
   }
   else
   {
       printf("a 大于 20\n" );
   } 
   return 0;
}
```

## switch 语句

```c
#include <stdio.h>
int main()
{
    char grade;
    printf("Please enter the grade:");
    scanf("%c", &grade);
    switch (grade)
    {
    case 'A':
        printf("很棒！\n");
        break;
    case 'B':
    case 'C':
        printf("做得好！\n");
        break;
    case 'D':
        printf("您通过了！\n");
        break;
    case 'E':
        printf("最好再试一下\n");
        break;
    default:
        printf("无效的成绩\n");
    }
    return 0;
}
```

## ? : 运算符(三元运算符)

```c
Exp1 ? Exp2 : Exp3;
```

如果 Exp1 为真，则计算 Exp2 的值。
如果 Exp1 为假，则计算 Exp3 的值。

```c
#include<stdio.h>
 
int main()
{
    int num;
 
    printf("输入一个数字 : ");
    scanf("%d",&num);
 
    (num%2==0)?printf("偶数"):printf("奇数");
}
```