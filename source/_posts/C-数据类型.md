---
title: C++-数据类型
date: 2021-06-23 11:28:08
tags:
- C语言
- CPP
- VSCode
- VS
- MingGW64
categories: CPP
---

## 内置类型

| 类型	| 关键字 |
|:---|:---|
|布尔型	| bool |
|字符型	| char |
|整型	| int |
|浮点型	| float |
|双浮点型	| double |
|无类型	| void |
|宽字符型 | wchar_t |

`wchar_t` 定义：

```cpp
typedef short int wchar_t;
```

`wchar_t` 实际上的空间是和 `short int` 一样。

一些基本类型可以使用一个或多个类型修饰符进行修饰：

* signed
* unsigned
* short
* long
<!--more-->

**注意：**不同系统会有所差异，一字节为 8 位。

**注意：**`long int 8` 个字节，`int` 都是 4 个字节，早期的 C 编译器定义了 `long int` 占用 4 个字节，`int` 占用 2 个字节，新版的 `C/C++` 标准兼容了早期的这一设定。

|类型	| 位  |	范围 |
|:---  | :--- | :--- |
|char	|1 个字节	| -128 到 127 或者 0 到 255|
|unsigned char|	1 个字节 |	0 到 255 |
|signed char |	1 个字节	| -128 到 127 |
|int |	4 个字节 |	-2147483648 到 2147483647 |
|unsigned int |	4 个字节 |	0 到 4294967295 |
|signed int	| 4 个字节	| -2147483648 到 2147483647 |
|short int	| 2 个字节 |	-32768 到 32767 |
|unsigned short int	| 2 个字节	| 0 到 65,535 |
|signed short int	| 2 个字节 |	-32768 到 32767 |
|long int	| 8 个字节 |	-9,223,372,036,854,775,808 到 9,223,372,036,854,775,807 |
|signed long int |	8 个字节	| -9,223,372,036,854,775,808 到 9,223,372,036,854,775,807|
|unsigned long int |	8 个字节 |	0 到 18,446,744,073,709,551,615 |
|float |	4 个字节 |	精度型占4个字节（32位）内存空间，+/- 3.4e +/- 38 (~7 个数字)|
|double	 |8 个字节 |	双精度型占8 个字节（64位）内存空间，+/- 1.7e +/- 308 (~15 个数字) | 
|long double |	16 个字节	| 长双精度型 16 个字节（128位）内存空间，可提供18-19位有效数字。|
|wchar_t	| 2 或 4 个字节	|1 个宽字符 |

实例：

`endl`: 在每一行后插入一个换行符;
`<<`: 运算符用于向屏幕传多个值;
`sizeof()`: 函数用来获取各种数据类型的大小。

```cpp
#include<iostream>  
#include <limits>
 
using namespace std;  
  
int main()  
{  
    cout << "type: \t\t" << "************size**************"<< endl;  
    cout << "bool: \t\t" << "所占字节数：" << sizeof(bool);  
    cout << "\t最大值：" << (numeric_limits<bool>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<bool>::min)() << endl;  
    cout << "char: \t\t" << "所占字节数：" << sizeof(char);  
    cout << "\t最大值：" << (numeric_limits<char>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<char>::min)() << endl;  
    cout << "signed char: \t" << "所占字节数：" << sizeof(signed char);  
    cout << "\t最大值：" << (numeric_limits<signed char>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<signed char>::min)() << endl;  
    cout << "unsigned char: \t" << "所占字节数：" << sizeof(unsigned char);  
    cout << "\t最大值：" << (numeric_limits<unsigned char>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<unsigned char>::min)() << endl;  
    cout << "wchar_t: \t" << "所占字节数：" << sizeof(wchar_t);  
    cout << "\t最大值：" << (numeric_limits<wchar_t>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<wchar_t>::min)() << endl;  
    cout << "short: \t\t" << "所占字节数：" << sizeof(short);  
    cout << "\t最大值：" << (numeric_limits<short>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<short>::min)() << endl;  
    cout << "int: \t\t" << "所占字节数：" << sizeof(int);  
    cout << "\t最大值：" << (numeric_limits<int>::max)();  
    cout << "\t最小值：" << (numeric_limits<int>::min)() << endl;  
    cout << "unsigned: \t" << "所占字节数：" << sizeof(unsigned);  
    cout << "\t最大值：" << (numeric_limits<unsigned>::max)();  
    cout << "\t最小值：" << (numeric_limits<unsigned>::min)() << endl;  
    cout << "long: \t\t" << "所占字节数：" << sizeof(long);  
    cout << "\t最大值：" << (numeric_limits<long>::max)();  
    cout << "\t最小值：" << (numeric_limits<long>::min)() << endl;  
    cout << "unsigned long: \t" << "所占字节数：" << sizeof(unsigned long);  
    cout << "\t最大值：" << (numeric_limits<unsigned long>::max)();  
    cout << "\t最小值：" << (numeric_limits<unsigned long>::min)() << endl;  
    cout << "double: \t" << "所占字节数：" << sizeof(double);  
    cout << "\t最大值：" << (numeric_limits<double>::max)();  
    cout << "\t最小值：" << (numeric_limits<double>::min)() << endl;  
    cout << "long double: \t" << "所占字节数：" << sizeof(long double);  
    cout << "\t最大值：" << (numeric_limits<long double>::max)();  
    cout << "\t最小值：" << (numeric_limits<long double>::min)() << endl;  
    cout << "float: \t\t" << "所占字节数：" << sizeof(float);  
    cout << "\t最大值：" << (numeric_limits<float>::max)();  
    cout << "\t最小值：" << (numeric_limits<float>::min)() << endl;  
    cout << "size_t: \t" << "所占字节数：" << sizeof(size_t);  
    cout << "\t最大值：" << (numeric_limits<size_t>::max)();  
    cout << "\t最小值：" << (numeric_limits<size_t>::min)() << endl;  
    cout << "string: \t" << "所占字节数：" << sizeof(string) << endl;  
    // << "\t最大值：" << (numeric_limits<string>::max)() << "\t最小值：" << (numeric_limits<string>::min)() << endl;  
    cout << "type: \t\t" << "************size**************"<< endl;  
    return 0;  
}
```

输出：

```cpp
type:           ************size**************
bool:           所占字节数：1   最大值：1               最小值：0
char:           所占字节数：1   最大值：               最小值：€
signed char:    所占字节数：1   最大值：               最小值：€
unsigned char:  所占字节数：1   最大值：               最小值：
wchar_t:        所占字节数：2   最大值：65535           最小值：0
short:          所占字节数：2   最大值：32767           最小值：-32768
int:            所占字节数：4   最大值：2147483647      最小值：-2147483648
unsigned:       所占字节数：4   最大值：4294967295      最小值：0
long:           所占字节数：4   最大值：2147483647      最小值：-2147483648
unsigned long:  所占字节数：4   最大值：4294967295      最小值：0
double:         所占字节数：8   最大值：1.79769e+308    最小值：2.22507e-308
long double:    所占字节数：16  最大值：1.18973e+4932   最小值：3.3621e-4932
float:          所占字节数：4   最大值：3.40282e+38     最小值：1.17549e-38
size_t:         所占字节数：8   最大值：18446744073709551615    最小值：0
string:         所占字节数：32
type:           ************size**************
```

## typedef声明

`typedef` 为一个已有的类型取一个新的名字。语法：

```cpp
typedef type newname; 
```

实例：

```cpp
// 告诉编译器，feet 是 int 的另一个名称
typedef int feet;

// 声明变量
feet distance;
```

## 枚举类型

枚举类型(enumeration)是 C++ 中的一种派生数据类型，它是由用户定义的若干枚举常量的集合。

创建枚举，需要使用关键字 enum。枚举类型的一般形式为：

```cpp
enum 枚举名{ 
     标识符[=整型常数], 
     标识符[=整型常数], 
... 
    标识符[=整型常数]
} 枚举变量;
```

如果枚举没有初始化, 即省掉 `=整型常数` 时, 则从第一个标识符开始。

默认情况下，第一个名称的值为 0，第二个名称的值为 1，第三个名称的值为 2，以此类推。

```cpp
enum color { red, green, blue } c;
c = blue;
```

```cpp
enum color { red, green=5, blue };
```

`green` 的值为 5，`blue` 的值为 6，因为默认情况下，每个名称都会比它前面一个名称大 1，但 `red` 的值依然为 0。

[C++ 数据类型](https://www.runoob.com/cplusplus/cpp-data-types.html)