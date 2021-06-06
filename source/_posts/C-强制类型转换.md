---
title: C-强制类型转换
date: 2021-06-06 13:38:13
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

强制类型转换是把变量从一种类型转换为另一种数据类型。

实例:

使用强制类型转换运算符把一个整数变量除以另一个整数变量，得到一个浮点数：

```c
#include <stdio.h>
 
int main()
{
   int sum = 17, count = 5;
   double mean;
 
   mean = (double) sum / count;
   printf("Value of mean : %f\n", mean ); 
}
```

<!--more-->
结果：

```c
Value of mean : 3.400000
```

注意的是强制类型转换运算符的优先级大于除法，因此 `sum` 的值首先被转换为 `double` 型，然后除以 `count`，得到一个类型为 `double` 的值。

## 整数提升

整数提升是指把小于 `int` 或 `unsigned int` 的整数类型转换为 `int` 或 `unsigned int` 的过程。

实例:

```c
#include <stdio.h>
 
int main()
{
   int  i = 17;
   char c = 'c'; /* ascii 值是 99 */
   int sum;
 
   sum = i + c;
   printf("Value of sum : %d\n", sum );
}
```

结果：

```c
Value of sum : 116
```

sum 的值为 116，因为编译器进行了整数提升，在执行实际加法运算时，把 `c` 的值转换为对应的 `ascii` 值。

## 常用的算术转换

常用的算术转换是隐式地把值强制转换为相同的类型。编译器首先执行整数提升，如果操作数类型不同，则它们会被转换为下列层次中出现的最高层次的类型：

![usual_arithmetic_conversion.png](/img/usual_arithmetic_conversion.png)

![img1.png](/img/img1.png)

常用的算术转换不适用于赋值运算符、逻辑运算符 `&&` 和 `||`。

实例

```c
#include <stdio.h>
 
int main()
{
   int  i = 17;
   char c = 'c'; /* ascii 值是 99 */
   float sum;
 
   sum = i + c;
   printf("Value of sum : %f\n", sum );
 
}
```

结果：

```c
Value of sum : 116.000000
```

在这里，c 首先被转换为整数，但是由于最后的值是 float 型的，所以会应用常用的算术转换，编译器会把 i 和 c 转换为浮点型，并把它们相加得到一个浮点数。

注意：C 语言中 `printf` 输出 `double` 和 `float` 都可以用 `%f` 占位符 可以混用，而 `double` 可以额外用 `%lf`。

而 `scanf` 输入情况下 `double` 必须用 `%lf`，`float` 必须用 `%f` 不能混用。

[C 强制类型转换](https://www.runoob.com/cprogramming/c-type-casting.html)