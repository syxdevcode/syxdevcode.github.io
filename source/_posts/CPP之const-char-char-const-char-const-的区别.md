---
title: CPP之const char*, char const*, char*const 的区别
date: 2021-06-28 13:50:26
tags:
- C语言
- CPP
- VSCode
- VS
- MingGW64
categories: CPP
---

Bjarne 在他的 `The C++ Programming Language` 里面给出过一个助记的方法：**把一个声明从右向左读**。

```cpp
char * const cp; ( * 读成 pointer to ) 
cp is a const pointer to char 

const char * p; 
p is a pointer to const char; 

char const * p; 
```

同上因为`C++`里面没有`const*`的运算符，所以`const`只能属于前面的类型。

`C++`标准规定，`const`关键字放在类型或变量名之前等价的。

<!--more-->
```cpp
const int n=5;    //same as below
int const m=10;

const int *p;    //same as below  const (int) * p
int const *q;    // (int) const *p


char ** p1; 
//    pointer to    pointer to    char

const char **p2;
//    pointer to    pointer to const char

char * const * p3;
//    pointer to const pointer to    char

const char * const * p4;
//    pointer to const pointer to const char 

char ** const p5;
// const pointer to    pointer to    char 

const char ** const p6;
// const pointer to    pointer to const char 

char * const * const p7;
// const pointer to const pointer to    char 

const char * const * const p8;
// const pointer to const pointer to const char
```

说到这里，我们可以看一道以前Google的笔试题：

```cpp
const char *p="hello";       
foo(&p);  // 函数foo(const char **pp)下面说法正确的是［］
```

* A.函数foo()不能改变p指向的字符串内容。
* B.函数foo()不能使指针p指向malloc生成的地址。
* C.函数foo()可以使p指向新的字符串常量。
* D.函数foo()可以把p赋值为 NULL。

至于这道题的答案是众说纷纭。针对上面这道题，我们可以用下面的程序测试：

```cpp
#include <stdio.h>
#include <stdlib.h>
#include <stdio.h>
void foo(const char **pp)
{
//    *pp=NULL;
//    *pp="Hello world!";
        *pp = (char *) malloc(10);
        snprintf(*pp, 10, "hi google!");
//       (*pp)[1] = 'x';
}
int
main()
{
    const char *p="hello";
    printf("before foo %s/n",p);
    foo(&p);
    printf("after foo %s/n",p);
    p[1] = 'x';
    return;
}
```

结论如下：

* 在foo函数中，可以使main函数中p指向的新的字符串常量。
* 在foo函数中，可以使main函数中的p指向NULL。
* 在foo函数中，可以使main函数中的p指向由`malloc`生成的内存块，并可以在`main`中用`free`释放，但是会有警告。但是注意，即使在foo中让p指向了由`malloc`生成的内存块，但是仍旧不能用`p[1]='x'`;这样的语句改变p指向的内容。
* 在foo中，不能用`(*pp)[1]='x'`;这样的语句改变p的内容。

所以，感觉`gcc`只是根据`const`的字面的意思对其作了限制，即对于 `const char*p` 这样的指针，不管后来p实际指向 `malloc` 的内存或者常量的内存，均不能用 `p[1]='x'` 这样的语句改变其内容。但是很奇怪，在foo里面，对p指向`malloc`的内存后，可以用 `snprintf` 之类的函数修改其内容。

[const char*, char const*, char*const 的区别](https://www.runoob.com/w3cnote/const-char.html)