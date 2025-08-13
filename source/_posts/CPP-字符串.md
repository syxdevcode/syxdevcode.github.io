---
title: CPP-字符串
date: 2021-06-29 17:08:56
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---

C++ 提供了以下两种类型的字符串表示形式：

* C 风格字符串
* C++ 引入的 string 类类型

<!--more-->
## C 风格字符串

C 风格的字符串起源于 C 语言，并在 C++ 中继续得到支持。字符串实际上是使用 `null` 字符 `\0` 终止的一维字符数组。因此，一个以 `null` 结尾的字符串，包含了组成字符串的字符。C++ 编译器会在初始化数组时，自动把 `\0` 放在字符串的末尾。

```cpp
char site[7] = {'R', 'U', 'N', 'O', 'O', 'B', '\0'};

// 等同于
// C++ 编译器会在初始化数组时，自动把 \0 放在字符串的末尾。
char site[] = "RUNOOB";
```

C++ 中有大量的函数用来操作以 `null` 结尾的字符串:

```cpp
strcpy(s1, s2)：复制字符串 s2 到字符串 s1。(不安全)
strcpy_s(s1, s2)：复制字符串 s2 到字符串 s1。(安全)
strcat(s1, s2)：连接字符串 s2 到字符串 s1 的末尾。连接字符串也可以用 + 号，

例如:
string str1 = "runoob";
string str2 = "google";
string str = str1 + str2;

strlen(s1)：返回字符串 s1 的长度。
strcmp(s1, s2)：如果 s1 和 s2 是相同的，则返回 0；如果 s1<s2 则返回值小于 0；如果 s1>s2 则返回值大于 0。如果两个字符串不匹配，且第一个字符串的第一个字符小于第二个字符串的第一个字符，才会返回比0小的值，反之则返回比0大的值。
strchr(s1, ch)：返回一个指针，指向字符串 s1 中字符 ch 的第一次出现的位置。
strstr(s1, s2)：返回一个指针，指向字符串 s1 中字符串 s2 的第一次出现的位置。
append()：在字符串的末尾添加字符
find()：在字符串中查找字符串
insert()：插入字符
length()：返回字符串的长度
replace()：替换字符串
substr()：返回某个子字符串
```

实例：

```cpp
#include <iostream>
#include <cstring>

using namespace std;

int main()
{
    char str1[13] = "runoob";
    char str2[13] = "google";
    char str3[13];
    int len;

    // 复制 str1 到 str3
    strcpy(str3, str1);
    cout << "strcpy( str3, str1) : " << str3 << endl;

    // 连接 str1 和 str2
    strcat(str1, str2);
    cout << "strcat( str1, str2): " << str1 << endl;

    // 连接后，str1 的总长度
    len = strlen(str1);
    cout << "strlen(str1) : " << len << endl;

    return 0;
}
```

结果：

```cpp
strcpy( str3, str1) : runoob
strcat( str1, str2): runoobgoogle
strlen(str1) : 12
```

## String 类

C++ 标准库提供了 string 类类型。

```cpp
#include <iostream>
#include <string>

using namespace std;

int main()
{
    string str1 = "runoob";
    string str2 = "google";
    string str3;
    int len;

    // 复制 str1 到 str3
    str3 = str1;
    cout << "str3 : " << str3 << endl;

    // 连接 str1 和 str2
    str3 = str1 + str2;
    cout << "str1 + str2 : " << str3 << endl;

    // 连接后，str3 的总长度
    len = str3.size();
    cout << "str3.size() :  " << len << endl;

    return 0;
}
```






