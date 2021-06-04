---
title: C-函数指针与回调函数
date: 2021-06-02 10:46:08
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

## 函数指针

函数指针是指向函数的指针变量。

函数指针可以像一般函数一样，用于调用函数、传递参数。

函数指针变量的声明：

```c
typedef int (*fun_ptr)(int,int); //声明一个指向同样参数、返回值的函数指针类型
```

<!--more-->
实例:

声明了函数指针变量 `p`，指向函数 `max`

```c
#include <stdio.h>

int max(int x, int y)
{
    return x > y ? x : y;
}

int main(void)
{
    /* p 是函数指针 */
    int (*p)(int, int) = &max; // &可以省略
    int a, b, c, d;

    printf("请输入三个数字:");
    scanf("%d %d %d", &a, &b, &c);

    /* 与直接调用函数等价，d = max(max(a, b), c) */
    d = p(p(a, b), c);

    printf("最大的数字是: %d\n", d);

    return 0;
}
```

结果如下：

```c
请输入三个数字:1 2 3
最大的数字是: 3
```

## 回调函数

函数指针作为某个函数的参数

函数指针变量可以作为某个函数的参数来使用的，回调函数就是一个通过函数指针调用的函数。

简单讲：回调函数是由别人的函数执行时调用你实现的函数。

实例

实例中 `populate_array` 函数定义了三个参数，其中第三个参数是函数的指针，通过该函数来设置数组的值。

实例中我们定义了回调函数 `getNextRandomValue`，它返回一个随机值，它作为一个函数指针传递给 `populate_array` 函数。

`populate_array` 将调用 `10` 次回调函数，并将回调函数的返回值赋值给数组。

实例:

```c
#include <stdlib.h>  
#include <stdio.h>
 
// 回调函数
void populate_array(int *array, size_t arraySize, int (*getNextValue)(void))
{
    for (size_t i=0; i<arraySize; i++)
        array[i] = getNextValue();
}
 
// 获取随机值
int getNextRandomValue(void)
{
    return rand();
}
 
int main(void)
{
    int myarray[10];
    /* getNextRandomValue 不能加括号，否则无法编译，因为加上括号之后相当于传入此参数时传入了 int , 而不是函数指针*/
    populate_array(myarray, 10, getNextRandomValue);
    for(int i = 0; i < 10; i++) {
        printf("%d ", myarray[i]);
    }
    printf("\n");
    return 0;
}
```

编译执行，输出结果如下：

```c
41 18467 6334 26500 19169 15724 11478 29358 26962 24464 
```

关于 `size_t`:

`size_t` 是一种数据类型，近似于无符号整型，但容量范围一般大于 `int` 和 `unsigned`。这里使用 `size_t` 是为了保证 `arraysize` 变量能够有足够大的容量来储存可能大的数组。

但凡不涉及负值范围的表示 `size` 取值的，都可以用 `size_t`；比如 `array[size_t]`。
`size_t` 在 `stddef.h` 头文件中定义。

在其他常见的宏定义以及函数中常用到有：

* 1，sizeof运算符返回的结果是size_t类型；
* 2，void *malloc(size_t size)...

参数：

[C-函数指针与回调函数](https://www.runoob.com/cprogramming/c-fun-pointer-callback.html)