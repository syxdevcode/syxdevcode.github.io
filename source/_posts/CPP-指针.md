---
title: CPP-指针
date: 2021-06-29 17:30:25
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

每一个变量都有一个内存位置，每一个内存位置都定义了可使用连字号（&）运算符访问的地址，它表示了在内存中的一个地址。

```cpp
#include <iostream>

using namespace std;

int main()
{
    int var1;
    char var2[10];

    cout << "var1 变量的地址： ";
    cout << &var1 << endl;

    cout << "var2 变量的地址： ";
    cout << &var2 << endl;

    return 0;
}
```

结果：

```cpp
var1 变量的地址： 0x61fe1c
var2 变量的地址： 0x61fe12
```

<!--more-->
## 指针

指针是一个变量，其值为另一个变量的地址，即，内存位置的直接地址。就像其他变量或常量一样，必须在使用指针存储其他变量地址之前，对其进行声明。指针变量声明的一般形式为：

```cpp
type *var-name;
```

在这里，type 是指针的基类型，它必须是一个有效的 C++ 数据类型，`var-name` 是指针变量的名称。用来声明指针的星号 `*` 与乘法中使用的星号是相同的。但是，在这个语句中，星号是用来指定一个变量是指针。

以下是有效的指针声明：

```cpp
int    *ip;    /* 一个整型的指针 */
double *dp;    /* 一个 double 型的指针 */
float  *fp;    /* 一个浮点型的指针 */
char   *ch;    /* 一个字符型的指针 */
```

所有指针的值的实际数据类型，不管是整型、浮点型、字符型，还是其他的数据类型，都是一样的，都是一个代表内存地址的长的十六进制数。不同数据类型的指针之间唯一的不同是，指针所指向的变量或常量的数据类型不同。

## 使用指针

使用指针时会频繁进行以下几个操作：定义一个指针变量、把变量地址赋值给指针、访问指针变量中可用地址的值。这些是通过使用一元运算符 * 来返回位于操作数所指定地址的变量的值。

实例：

```cpp
#include <iostream>

using namespace std;

int main()
{
    int var = 20; // 实际变量的声明
    int *ip;      // 指针变量的声明

    ip = &var; // 在指针变量中存储 var 的地址

    cout << "Value of var variable: ";
    cout << var << endl;

    // 输出在指针变量中存储的地址
    cout << "Address stored in ip variable: ";
    cout << ip << endl;

    // 访问指针中地址的值
    cout << "Value of *ip variable: ";
    cout << *ip << endl;

    return 0;
}
```

结果：

```cpp
Value of var variable: 20
Address stored in ip variable: 0x61fe14
Value of *ip variable: 20
```

## 空指针

在变量声明的时候，如果没有确切的地址可以赋值，为指针变量赋一个 空值是一个良好的编程习惯。赋为 空值的指针被称为空指针。

C中的`NULL`，在C头文件中，通常定义如下：

```c
#define NULL ((void*)0)
```

但是在C++中，它是这样定义的：

```cpp
#define NULL 0
```

原因是C++中不能将`void *`类型的指针隐式转换成其他指针类型。

```cpp
#include<iostream>
int main(void)
{
    char p[] = "12345";
    int *a = (void*)p;
    return 0;
}
```

报错：

```cpp
null.cpp:5:17: error: invalid conversion from 'void*' to 'int*' [-fpermissive]
  int *a =(void*)p;
```

**nullptr**

C++11标准后，用`nullptr`来表示空指针。

实例：

```cpp
#include<iostream>
using namespace std;
void test(void *p)
{
    cout<<"p is pointer "<<p<<endl;
}
void test(int num)
{
    cout<<"num is int "<<num<<endl;
}
int main(void)
{
    // 报错：error: call of overloaded ‘test(NULL)’ is ambiguous test(NULL);
    test(NULL);

    // 不会报错
    test(nullptr);
    return 0;
}
```

[为什么建议你用nullptr而不是NULL？](https://cloud.tencent.com/developer/article/1494848)

## 指针的算术运算

指针是一个用数值表示的地址。可以对指针执行算术运算。可以对指针进行四种算术运算：`++、--、+、-`。


假设 ptr 是一个指向地址 1000 的整型指针，是一个 32 位的整数，执行 `ptr++` 运算之后，`ptr` 将指向位置 1004，因为 `ptr` 每增加一次，它都将指向下一个整数位置，即当前位置往后移 4 个字节。这个运算会在不影响内存位置中实际值的情况下，移动指针到下一个内存位置。

如果 `ptr` 指向一个地址为 1000 的字符，上面的运算会导致指针指向位置 1001，因为下一个字符位置是在 1001。

### 递增一个指针

对指针进行递增运算，即把值加上其数据类型的字节数。

在程序中使用指针代替数组，因为变量指针可以递增，而数组不能递增，因为数组是一个常量指针。

下面的程序递增变量指针，以便顺序访问数组中的每一个元素：

```cpp
#include <iostream>

using namespace std;
const int MAX = 3;

int main()
{
    int var[MAX] = {10, 100, 200};
    int *ptr;

    // 指针中的数组地址
    ptr = var;
    for (int i = 0; i < MAX; i++)
    {
        cout << "Address of var[" << i << "] = ";
        cout << ptr << endl;

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
Address of var[0] = 0xbfa088b0
Value of var[0] = 10
Address of var[1] = 0xbfa088b4
Value of var[1] = 100
Address of var[2] = 0xbfa088b8
Value of var[2] = 200
```

### 递减一个指针

对指针进行递减运算，即把值减去其数据类型的字节数。

实例：

```cpp
#include <iostream>

using namespace std;
const int MAX = 3;

int main()
{
    int var[MAX] = {10, 100, 200};
    int *ptr;

    // 指针中最后一个元素的地址
    ptr = &var[MAX - 1];
    for (int i = MAX; i > 0; i--)
    {
        cout << "Address of var[" << i << "] = ";
        cout << ptr << endl;

        cout << "Value of var[" << i << "] = ";
        cout << *ptr << endl;

        // 移动到下一个位置
        ptr--;
    }
    return 0;
}
```

结果：

```cpp
Address of var[3] = 0xbfdb70f8
Value of var[3] = 200
Address of var[2] = 0xbfdb70f4
Value of var[2] = 100
Address of var[1] = 0xbfdb70f0
Value of var[1] = 10
```

### 指针的比较

指针可以用关系运算符进行比较，如 `==`、`<` 和 `>`。
如果 p1 和 p2 指向两个相关的变量，比如同一个数组中的不同元素，则可对 p1 和 p2 进行大小比较。

实例，只要变量指针所指向的地址小于或等于数组的最后一个元素的地址 `&var[MAX - 1]`，则把变量指针进行递增：

```cpp
#include <iostream>

using namespace std;
const int MAX = 3;

int main()
{
    int var[MAX] = {10, 100, 200};
    int *ptr;

    // 指针中第一个元素的地址
    ptr = var;
    int i = 0;
    while (ptr <= &var[MAX - 1])
    {
        cout << "Address of var[" << i << "] = ";
        cout << ptr << endl;

        cout << "Value of var[" << i << "] = ";
        cout << *ptr << endl;

        // 指向上一个位置
        ptr++;
        i++;
    }
    return 0;
}
```

结果：

```cpp
Address of var[0] = 0xbfce42d0
Value of var[0] = 10
Address of var[1] = 0xbfce42d4
Value of var[1] = 100
Address of var[2] = 0xbfce42d8
Value of var[2] = 200
```

## 指针与数组

指针和数组是密切相关的。事实上，指针和数组在很多情况下是可以互换的。例如，一个指向数组开头的指针，可以通过使用指针的算术运算或数组索引来访问数组。

实例：

```cpp
#include <iostream>

using namespace std;
const int MAX = 3;

int main()
{
    int var[MAX] = {10, 100, 200};
    int *ptr;

    // 指针中的数组地址
    ptr = var;
    for (int i = 0; i < MAX; i++)
    {
        cout << "var[" << i << "] ";
        cout << ptr << endl;

        cout << "var[" << i << "] ";
        cout << *ptr << endl;

        // 移动到下一个位置
        ptr++;
    }
    return 0;
}
```

结果：

```cpp
var[0]的内存地址为 0x7fff59707adc
var[0] 的值为 10
var[1]的内存地址为 0x7fff59707ae0
var[1] 的值为 100
var[2]的内存地址为 0x7fff59707ae4
var[2] 的值为 200
```

## 指针数组

可以定义用来存储指针的数组。

```cpp
#include <iostream>

using namespace std;
const int MAX = 3;

int main()
{
    int var[MAX] = {10, 100, 200};
    int *ptr[MAX];

    const char *names[MAX] = {
        "Zara Ali",
        "Hina Ali",
        "Nuha Ali",
    };

    for (int i = 0; i < MAX; i++)
    {
        ptr[i] = &var[i]; // 赋值为整数的地址
    }
    for (int i = 0; i < MAX; i++)
    {
        cout << "Value of var[" << i << "] = ";
        cout << *ptr[i] << endl;
        cout << "char:" << names[i] << endl;
    }
    return 0;
}
```

结果：

```cpp
Value of var[0] = 10
char:Zara Ali
Value of var[1] = 100
char:Hina Ali
Value of var[2] = 200
char:Nuha Ali
```

## 指向指针的指针（多级间接寻址）

指向指针的指针是一种多级间接寻址的形式，或者说是一个指针链。

指针的指针就是将指针的地址存放在另一个指针里面。

通常，一个指针包含一个变量的地址。当我们定义一个指向指针的指针时，第一个指针包含了第二个指针的地址，第二个指针指向包含实际值的位置。

![pointer_to_pointer.jpg](/img/pointer_to_pointer.jpg)

实例：

```cpp
#include <iostream>

using namespace std;

int main()
{
    int var;
    int *ptr;
    int **pptr;

    var = 3000;

    // 获取 var 的地址
    ptr = &var;

    // 使用运算符 & 获取 ptr 的地址
    pptr = &ptr;

    // 使用 pptr 获取值
    cout << "var  :" << var << endl;
    cout << "*ptr :" << *ptr << endl;
    cout << "**pptr :" << **pptr << endl;

    return 0;
}
```

结果：

```cpp
var  :3000
*ptr :3000
**pptr :3000
```

## 传递指针给函数

通过引用或地址传递参数，使传递的参数在调用函数中被改变。

C++ 允许您传递指针给函数，只需要简单地声明函数参数为指针类型即可。

实例：

```cpp
#include <iostream>
#include <ctime>

using namespace std;

// 在写函数时应习惯性的先声明函数，然后在定义函数
void getSeconds(unsigned long *par);

int main()
{
    unsigned long sec;

    getSeconds(&sec);

    // 输出实际值
    cout << "Number of seconds :" << sec << endl;

    return 0;
}

void getSeconds(unsigned long *par)
{
    // 获取当前的秒数
    *par = time(nullptr);
    return;
}
```

结果：

```cpp
Number of seconds :1625019462
```

接受数组作为参数:

```cpp
#include <iostream>
using namespace std;

// 函数声明
double getAverage(int *arr, int size);

int main()
{
    // 带有 5 个元素的整型数组
    int balance[5] = {1000, 2, 3, 17, 50};
    double avg;

    // 传递一个指向数组的指针作为参数
    avg = getAverage(balance, 5);

    // 输出返回值
    cout << "Average value is: " << avg << endl;

    return 0;
}

double getAverage(int *arr, int size)
{
    int i, sum = 0;
    double avg;

    for (i = 0; i < size; ++i)
    {
        sum += arr[i];
    }

    avg = double(sum) / size;

    return avg;
}
```

结果：

```cpp
Average value is: 214.4
```

## 从函数返回指针

C++ 允许函数返回指针到局部变量、静态变量和动态内存分配。


声明一个返回指针的函数：

```cpp
int * myFunction()
{
}
```

注意：C++ 不支持在函数外返回局部变量的地址，除非定义局部变量为 `static` 变量。

实例：

生成 10 个随机数，并使用表示指针的数组名（即第一个数组元素的地址）来返回

```cpp
#include <iostream>
#include <ctime>
#include <cstdlib>

using namespace std;

// 要生成和返回随机数的函数
int *getRandom()
{
    static int r[10];

    // 设置种子
    srand((unsigned)time(nullptr));
    for (int i = 0; i < 10; ++i)
    {
        r[i] = rand();
        cout << r[i] << endl;
    }

    return r;
}

// 要调用上面定义函数的主函数
int main()
{
    // 一个指向整数的指针
    int *p;

    p = getRandom();
    for (int i = 0; i < 10; i++)
    {
        cout << "*(p + " << i << ") : ";
        cout << *(p + i) << endl;
    }

    return 0;
}
```

结果：

```cpp
17074
22711
18431
31312
26191
20269
11507
11699
29611
17869
*(p + 0) : 17074
*(p + 1) : 22711
*(p + 2) : 18431
*(p + 3) : 31312
*(p + 4) : 26191
*(p + 5) : 20269
*(p + 6) : 11507
*(p + 7) : 11699
*(p + 8) : 29611
*(p + 9) : 17869
```