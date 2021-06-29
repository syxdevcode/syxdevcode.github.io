---
title: CPP-简介
date: 2021-06-08 22:07:41
tags:
- CPP
- VSCode
- MingGW64
categories: CPP
---
## 简介

`C++` 是一种静态类型的、编译式的、通用的、大小写敏感的、不规则的编程语言，支持过程化编程、面向对象编程和泛型编程。

`C++` 被认为是一种中级语言，它综合了高级语言和低级语言的特点。

`C++` 是 C 的一个超集，事实上，任何合法的 C 程序都是合法的 `C++` 程序。

注意：使用静态类型的编程语言是在编译时执行类型检查，而不是在运行时执行类型检查。

## 面向对象程序设计

`C++` 完全支持面向对象的程序设计，包括面向对象开发的四大特性：

* 封装
* 抽象
* 继承
* 多态

<!--more-->
## 标准库

标准的 `C++` 由三个重要部分组成：

* 核心语言，提供了所有构件块，包括变量、数据类型和常量，等等。
* `C++` 标准库，提供了大量的函数，用于操作文件、字符串等。
* 标准模板库（STL），提供了大量的方法，用于操作数据结构等。

## ANSI 标准

`ANSI` 标准是为了确保 `C++` 的便携性 —— 所编写的代码在 Mac、UNIX、Windows、Alpha 计算机上都能通过编译。

由于 ANSI 标准已稳定使用了很长的时间，所有主要的 `C++` 编译器的制造商都支持 ANSI 标准。

## 编辑器

通过编辑器创建的文件通常称为源文件，源文件包含程序源代码。`C++` 程序的源文件通常使用扩展名 `.cpp`、`.cp` 或 `.c`。

## C++ 编译器

大多数的 `C++` 编译器并不在乎源文件的扩展名，但是如果未指定扩展名，则默认使用 `.cpp`。

验证：

`main.cpp`：

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello World!\n";
    return 0;
}
```

## g++ 应用说明

程序 `g++` 是将 `gcc` 默认语言设为 `C++` 的一个特殊的版本，链接时它自动使用 `C++` 标准库而不用 C 标准库。通过遵循源码的命名规范并指定对应库的名字，用 `gcc` 来编译链接 `C++` 程序是可行的，如下例所示：

```c
$ gcc main.cpp -lstdc++ -o main
```

下面是一个保存在文件 `helloworld.cpp` 中一个简单的 `C++` 程序的代码：

```cpp
#include <iostream>
using namespace std;
int main()
{
    cout << "Hello, world!" << endl;
    return 0;
}
```

最简单的编译方式：

```cpp
$ g++ helloworld.cpp
```

由于命令行中未指定可执行程序的文件名，编译器采用默认的 `a.out`。程序可以这样来运行：

```cpp
$ ./a.out
Hello, world!
```

通常使用 -o 选项指定可执行程序的文件名，以下实例生成一个 helloworld 的可执行文件：

```cpp
$ g++ helloworld.cpp -o helloworld
```

执行 helloworld:

```cpp
$ ./helloworld
Hello, world!
```

如果是多个 `C++` 代码文件，如 `runoob1.cpp、runoob2.cpp`，编译命令如下：

```cpp
$ g++ runoob1.cpp runoob2.cpp -o runoob
```

生成一个 runoob 可执行文件。

`g++` 有些系统默认是使用 `C++98`，我们可以指定使用 `C++11` 来编译 `main.cpp` 文件：

```cpp
g++ -g -Wall -std=c++11 main.cpp
```

g++ 常用命令选项

| 选项	| 解释 |
| :--- | :--- |
|-ansi	|只支持 ANSI 标准的 C 语法。这一选项将禁止 GNU C 的某些特色， 例如 asm 或 typeof 关键词。|
|-c	|只编译并生成目标文件。|
|-DMACRO	|以字符串`"1"`定义 MACRO 宏。|
|-DMACRO=DEFN	|以字符串`"DEFN"`定义 MACRO 宏。|
|-E	|只运行 C 预编译器。|
|-g	|生成调试信息。GNU 调试器可利用该信息。|
|-IDIRECTORY|	指定额外的头文件搜索路径DIRECTORY。|
|-LDIRECTORY|	指定额外的函数库搜索路径DIRECTORY。|
|-lLIBRARY|	连接时搜索指定的函数库LIBRARY。|
|-m486	|针对 486 进行代码优化。|
|-o	|FILE 生成指定的输出文件。用在生成可执行文件时。|
|-O0|	不进行优化处理。|
|-O	或 -O1 |优化生成代码。|
|-O2|	进一步优化。|
|-O3	|比 -O2 更进一步优化，包括 inline 函数。|
|-shared	|生成共享目标文件。通常用在建立共享库时。|
|-static	|禁止使用共享连接。|
|-UMACRO	|取消对 MACRO 宏的定义。|
|-w	|不生成任何警告信息。|
|-Wall	|生成所有警告信息。|
