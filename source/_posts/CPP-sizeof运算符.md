---
title: CPP-sizeof运算符
date: 2021-06-28 16:53:16
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

`sizeof` 是一个关键字，它是一个编译时运算符，用于判断变量或数据类型的字节大小。

`sizeof` 运算符可用于获取类、结构、共用体和其他用户自定义数据类型的大小。

使用 `sizeof` 的语法如下：

```cpp
sizeof (data type)
```

其中，`data type` 是要计算大小的数据类型，包括类、结构、共用体和其他用户自定义数据类型。

实例：

```cpp
#include <iostream>
using namespace std;
 
int main()
{
   cout << "Size of char : " << sizeof(char) << endl;
   cout << "Size of int : " << sizeof(int) << endl;
   cout << "Size of short int : " << sizeof(short int) << endl;
   cout << "Size of long int : " << sizeof(long int) << endl;
   cout << "Size of float : " << sizeof(float) << endl;
   cout << "Size of double : " << sizeof(double) << endl;
   cout << "Size of wchar_t : " << sizeof(wchar_t) << endl;
   return 0;
}
```

结果，结果会根据使用的机器而不同：

```cpp
Size of char : 1
Size of int : 4
Size of short int : 2
Size of long int : 4
Size of float : 4
Size of double : 8
Size of wchar_t : 4
```

[C++ sizeof 运算符](https://www.runoob.com/cplusplus/cpp-sizeof-operator.html)