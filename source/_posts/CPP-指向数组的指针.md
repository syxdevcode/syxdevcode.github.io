---
title: CPP-指向数组的指针
date: 2021-06-29 16:25:04
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

数组名是指向数组中第一个元素的常量指针。

声明：

```cpp
double runoobAarray[50];
```

`runoobAarray` 是一个指向 `&runoobAarray[0]` 的指针，即数组 `runoobAarray` 的第一个元素的地址。
因此，下面的程序片段把 p 赋值为 `runoobAarray` 的第一个元素的地址：

```cpp
double *p;
double runoobAarray[10];

p = runoobAarray;
```

使用数组名作为常量指针是合法的，反之亦然。因此，`*(runoobAarray + 4)` 是一种访问 `runoobAarray[4]` 数据的合法方式。使用 `*p、*(p+1)、*(p+2)` 等来访问数组元素。

<!--more-->
```cpp
#include <iostream>
using namespace std;
 
int main ()
{
   // 带有 5 个元素的双精度浮点型数组
   double runoobAarray[5] = {1000.0, 2.0, 3.4, 17.0, 50.0};
   double *p;
 
   p = runoobAarray;
 
   // 输出数组中每个元素的值
   cout << "使用指针的数组值 " << endl; 
   for ( int i = 0; i < 5; i++ )
   {
       cout << "*(p + " << i << ") : ";
       cout << *(p + i) << endl;
   }
 
   cout << "使用 runoobAarray 作为地址的数组值 " << endl;
   for ( int i = 0; i < 5; i++ )
   {
       cout << "*(runoobAarray + " << i << ") : ";
       cout << *(runoobAarray + i) << endl;
   }
 
   return 0;
}
```

结果：

```cpp
使用指针的数组值 
*(p + 0) : 1000
*(p + 1) : 2
*(p + 2) : 3.4
*(p + 3) : 17
*(p + 4) : 50
使用 runoobAarray 作为地址的数组值 
*(runoobAarray + 0) : 1000
*(runoobAarray + 1) : 2
*(runoobAarray + 2) : 3.4
*(runoobAarray + 3) : 17
*(runoobAarray + 4) : 50
```

C++ 中，将 `char *` 或 `char[]` 传递给 `cout` 进行输出，结果会是整个字符串，如果想要获得字符串的地址（第一个字符的内存地址），需要强制转化为其他指针（非 `char *`）。可以是 `void *`，`int *`，`float *`，`double *` 等。

使用 `&s[0]` 不能输出 `s[0]`（首字符）的地址。因为 `&s[0]` 将返回 `char*`，对于 `char*`（char 指针），`cout` 会将其作为字符串来处理，向下查找字符并输出直到字符结束。

实例：

```cpp
#include <iostream>
 
using namespace std;
const int MAX = 3;
 
int main ()
{
   char  var[MAX] = {'a', 'b', 'c'};
   char  *ptr;
 
   // 指针中的数组地址
   ptr = var;
            
   for (int i = 0; i < MAX; i++)
   {

      cout << "Address of var[" << i << "] = ";
      cout << (int *)ptr << endl;
 
      cout << "Value of var[" << i << "] = ";
      cout << *ptr << endl;
 
      // 移动到下一个位置
      ptr++;
   }
   return 0;
}
```

结果：

```cpp
Address of var[0] = 0x61fe11
Value of var[0] = a
Address of var[1] = 0x61fe12
Value of var[1] = b
Address of var[2] = 0x61fe13
Value of var[2] = c
```