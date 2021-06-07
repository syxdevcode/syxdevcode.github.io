---
title: C-void*使用详解
date: 2021-06-06 21:47:55
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

## void 的作用

* 当函数不需要返回值时，必须使用void限定。例如：`void func（int a,char *b）`。
* 当函数不允许接受参数时，必须使用void限定。例如：`int func(void)`。

## void 指针的使用规则

1. `void` 指针可以指向任意类型的数据，就是说可以用任意类型的指针对 `void` 指针对 `void` 指针赋值。

例如：

```c
int *a；
void *p；
p=a;
```
<!--more-->

如果要将 `void` 指针 `p` 赋给其他类型的指针，则需要强制类型转换，就本例而言：`a=（int *）p`。

在内存的分配中我们可以见到 `void` 指针使用：内存分配函数 `malloc` 函数返回的指针就是 `void * `型，用户在使用这个指针的时候，要进行强制类型转换，也就是显式说明该指针指向的内存中是存放的什么类型的数据, `(int *)malloc(1024)` 表示强制规定 `malloc` 返回的 `void*` 指针指向的内存中存放的是一个的 `int` 型数据。

2. 在 `ANSI C` 标准中，不允许对 `void` 指针进行一些算术运算，如 `p++` 或 `p+=1` 等，因为既然 `void` 是无类型，那么每次算术运算我们就不知道该操作几个字节，例如 `char` 型操作 `sizeof(char)` 字节，而 `int` 则要操作 `sizeof(int)` 字节。而在 GNU 中则允许，因为在默认情况下，GNU 认为 `void *` 和 `char *` 一样，既然是确定的，当然可以进行一些算术操作，在这里`sizeof(*p)==sizeof(char)`。

`void` 几乎只有 `注释` 和限制程序的作用，因为从来没有人会定义一个 `void` 变量，让我们试着来定义：

```c
void a;
```

这行语句编译时会出错，提示 `illegal use of type 'void'`。即使 `void a` 的编译不会出错，它也没有任何实际意义。

如果指针 p1 和 p2 的类型相同，那么可以直接在 p1 和 p2 间互相赋值；如果 p1 和 p2 指向不同的数据类型，则必须使用强制类型转换运算符把赋值运算符右边的指针类型转换为左边指针的类型。

```c
float *p1;
int *p2;
p1 = p2;
//其中p1 = p2语句会编译出错，
//提示“'=' : cannot convert from 'int *' to 'float *'”，必须改为：
p1 = (float *)p2;
```

而 `void *` 则不同，任何类型的指针都可以直接赋值给它，无需进行强制类型转换。

```c
void *p1;
int *p2;
p1 = p2;
```

但这并不意味着，`void *` 也可以无需强制类型转换地赋给其它类型的指针。因为 `无类型` 可以包容 `有类型`，而 `有类型` 则不能包容 `无类型`。

**小心使用 void 指针类型:**

按照 `ANSI(American National Standards Institute)` 标准，不能对 `void` 指针进行算法操作，即下列操作都是不合法的：

```c
void * pvoid;
pvoid++; //ANSI：错误
pvoid += 1; //ANSI：错误
//ANSI标准之所以这样认定，是因为它坚持：进行算法操作的指针必须是确定知道其指向数据类型大小的。
//例如：
int *pint;

pint++; //ANSI：正确pint++ 的结果是使其增大 sizeof(int)。
```

但是 `GNU` 则不这么认定，它指定 `void *` 的算法操作与 `char *` 一致。因此下列语句在 GNU 编译器中皆正确：

```c
pvoid++; //GNU：正确
pvoid += 1; //GNU：正确
pvoid++ 的执行结果是其增大了 1。
```

在实际的程序设计中，为迎合 `ANSI` 标准，并提高程序的可移植性，我们可以这样编写实现同样功能的代码：

```c
void * pvoid;
((char *)pvoid)++; //ANSI：错误；GNU：正确
(char *)pvoid += 1; //ANSI：错误；GNU：正确
```

`GNU` 和 `ANSI` 还有一些区别，总体而言，`GNU` 较 `ANSI` 更"开放"，提供了对更多语法的支持。但是我们在真实设计时，还是应该尽可能地迎合 `ANSI` 标准。 如果函数的参数可以是任意类型指针，那么应声明其参数为`void *`。

注：`void` 指针可以任意类型的数据，可以在程序中给我们带来一些好处，函数中形为指针类型时，我们可以将其定义为 `void` 指针，这样函数就可以接受任意类型的指针。如：

典型的如内存操作函数 `memcpy` 和 `memset` 的函数原型分别为：

```c
void * memcpy(void *dest, const void *src, size_t len);
void * memset ( void * buffer, int c, size_t num );
```

这样，任何类型的指针都可以传入 `memcpy` 和 `memset` 中，这也真实地体现了内存操作函数的意义，因为它操作的对象仅仅是一片内存，而不论这片内存是什么类型（参见 [C 语言实现泛型编程](https://www.runoob.com/w3cnote/c-general-function.html)）。如果 `memcpy` 和 `memset` 的参数类型不是 `void *`，而是 `char *`，那才叫真的奇怪了！这样的 `memcpy` 和 `memset` 明显不是一个 `纯粹的，脱离低级趣味的` 函数！`void` 的出现只是为了一种抽象的需要，如果你正确地理解了面向对象中 `抽象基类` 的概念，也很容易理解 `void` 数据类型。正如不能给抽象基类定义一个实例，我们也不能定义一个 `void`（让我们类比的称 `void` 为 `抽象数据类型` ）变量。

[C 语言中 void* 详解及应用](https://www.runoob.com/w3cnote/c-void-intro.html)