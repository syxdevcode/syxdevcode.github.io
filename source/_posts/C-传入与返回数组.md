---
title: C-传入与返回数组
date: 2021-05-26 22:13:03
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

## 传递数组给函数

这三种类型，都是将整型指针传递给函数。

**方式 1**

形式参数是一个指针：

```c
void myFunction(int *param)
{
}
```

**方式 2**

形式参数是一个已定义大小的数组：

```c
void myFunction(int param[10])
{
}
```

**方式 3**

形式参数是一个未定义大小的数组：

```c
void myFunction(int param[])
{
}
```

<!--more-->
实例：

```c
#include <stdio.h>
 
/* 函数声明 */
double getAverage(int arr[], int size);
 
int main ()
{
   /* 带有 5 个元素的整型数组 */
   int balance[5] = {1000, 2, 3, 17, 50};
   double avg;
 
   /* 传递一个指向数组的指针作为参数 */
   avg = getAverage( balance, 5 ) ;
 
   /* 输出返回值 */
   printf( "平均值是： %f ", avg );
    
   return 0;
}
 
double getAverage(int arr[], int size)
{
  int    i;
  double avg;
  double sum=0;
 
  for (i = 0; i < size; ++i)
  {
    sum += arr[i];
  }
 
  avg = sum / size;
 
  return avg;
}
```

### 二维数组传递给函数

如果将二维数组作为实参传递给某个函数，第一维的长度可以不指定，但必须指定第二维的长度:

```c
double * MatrixMultiple(double a[][], double b[][]); /* 错误的 */

double * MatrixMultiple(double a[][2], double b[][3]);  /* 正确的 */
```

## 从函数返回数组

C 语言不允许返回一个完整的数组作为函数的参数。但是，您可以通过指定不带索引的数组名来返回一个指向数组的指针。

**实例**

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
 
/* 要生成和返回随机数的函数 */
int * getRandom( )
{
  static int  r[10];
  int i;
 
  /* 设置种子 */
  srand( (unsigned)time( NULL ) );
  for ( i = 0; i < 10; ++i)
  {
     r[i] = rand();
     printf( "r[%d] = %d\n", i, r[i]);
 
  }
 
  return r;
}
 
/* 要调用上面定义函数的主函数 */
int main ()
{
   /* 一个指向整数的指针 */
   int *p;
   int i;
 
   p = getRandom();
   for ( i = 0; i < 10; i++ )
   {
       printf( "*(p + %d) : %d\n", i, *(p + i));
   }
 
   return 0;
}
```

输出：

```c
r[0] = 11895
r[1] = 17345
r[2] = 27556
r[3] = 20904
r[4] = 1985
r[5] = 27000
r[6] = 17147
r[7] = 23148
r[8] = 2139
r[9] = 16697
*(p + 0) : 11895
*(p + 1) : 17345
*(p + 2) : 27556
*(p + 3) : 20904
*(p + 4) : 1985
*(p + 5) : 27000
*(p + 6) : 17147
*(p + 7) : 23148
*(p + 8) : 2139
*(p + 9) : 16697
```

解析：

`srand((unsigned)time(NULL))` 是初始化随机函数种子：

* 1、是拿当前系统时间作为种子，由于时间是变化的，种子变化，可以产生不相同的随机数。计算机中的随机数实际上都不是真正的随机数，如果两次给的种子一样，是会生成同样的随机序列的。 所以，一般都会以当前的时间作为种子来生成随机数，这样更加的随机。
* 2、使用时，参数可以是 `unsigned` 型的任意数据，比如 `srand（10）`；
* 3、如果不使用 `srand`，用 `rand()`产生的随机数，在多次运行，结果是一样的。