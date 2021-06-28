---
title: CPP-修饰符类型
date: 2021-06-28 14:11:25
tags:
- C语言
- CPP
- VSCode
- VS
- MingGW64
categories: CPP
---

C++ 允许在 `char`、`int` 和 `double` 数据类型前放置修饰符。修饰符用于改变基本类型的含义，所以它更能满足各种情境的需求。

下面列出了数据类型修饰符：

* signed
* unsigned
* long
* short

修饰符 `signed`、`unsigned`、`long` 和 `short` 可应用于整型，`signed` 和 `unsigned` 可应用于字符型，`long` 可应用于双精度型。

修饰符 `signed` 和 `unsigned` 也可以作为 `long` 或 `short` 修饰符的前缀。例如：`unsigned long int`。

C++ 允许使用速记符号来声明无符号短整数或无符号长整数。可以不写 `int`，只写单词 `unsigned`、`short` 或 `unsigned`、`long`，`int` 是隐含的。

例如，下面的两个语句都声明了无符号整型变量。

```cpp
unsigned x;
unsigned int y;
```

实例

```cpp
#include <iostream>
using namespace std;
 
/* 
 * 这个程序演示了有符号整数和无符号整数之间的差别
*/
int main()
{
   short int i;           // 有符号短整数
   short unsigned int j;  // 无符号短整数
 
   j = 50000;
 
   i = j;
   cout << i << " " << j;
 
   return 0;
}
```

结果：`-15536 50000`

上述结果中，无符号短整数 `50,000` 的位模式被解释为有符号短整数 `-15,536`。

对于无符号化为有符号的位数运算，采取 `N-2^n` 的计算方法，n 取决于定义的数据类型 `int`、`short`、`char`、`long int` 等等，`N` 为无符号数的数值，例如文中的 `N=50000`，`short` 为 16 位，计算方法为 `50000-2^16` 得到 `-15536`。

过程解析：

16 位整数（短整数）的情况下，十进制 `50000` 就是二进制 `11000011 01010000` 但在有符号的情况下，二进制最左边的 1，代表这整个数字是负数但是电脑是以补码形式来表示数字的，要获得原本的数字，首先要把整个二进制数 `-11100001101010000 - 1 = ‭1100001101001111`‬ 然后，在把答案取反码 `not ‭1100001101001111‬ = ‭0011110010110000`‬ 把最终答案变成十进制，就是 `15536` 所以，一开始的二进制数 `11000011 01010000`，在有符号的情况下代表的就是 `-15536`。

代码：

```cpp
#include <iostream>
#include <cstdlib>

using namespace std;

int main()
{
    short int i;           // 有符号短整数
    short unsigned int j;  // 无符号短整数

    j = 50000;
    i = j;
    cout <<"i:" <<i<<'\n'<< "j:" << j<<endl;
    char s[40];
    _itoa_s(i, s, 2);
    printf("变量i的二进制数为：%s\n", s);
    _itoa_s(j, s, 2);
    printf("变量j的二进制数为：%s\n", s);
    return 0;
}
```

结果为：

```cpp
i:-15536
j:50000
变量i的二进制数为：11111111111111111100001101010000
变量j的二进制数为：1100001101010000
```

## C++ 中的类型限定符

类型限定符提供了变量的额外信息。

* **const**：`const` 类型的对象在程序执行期间不能被修改改变。
* **volatile**：修饰符 `volatile` 告诉编译器不需要优化 `volatile` 声明的变量，让程序可以直接从内存中读取变量。对于一般的变量编译器会对变量进行优化，将内存中的变量值放在寄存器中以加快读写效率。
* **restrict**：由 `restrict` 修饰的指针是唯一一种访问它所指向的对象的方式。只有 `C99` 增加了新的类型限定符 `restrict`。

