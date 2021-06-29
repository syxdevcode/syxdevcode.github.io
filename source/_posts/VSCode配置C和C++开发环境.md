---
title: VSCode配置C和C++开发环境
date: 2021-05-06 21:15:32
tags:
- C语言
- CPP
- VSCode
- MingGW64
categories: C语言
---

## 安装编译器（MinGW-W64 GCC）

下载地址：[MinGW-w64 - for 32 and 64 bit Windows](https://sourceforge.net/projects/mingw-w64/files/)

![微信截图_20210506172354.png](/img/微信截图_20210506172354.png)

<!--more-->
## 配置环境

1，找到压缩文件所在位置，找到 `bin,include`文件夹，如下：

```c
D:\mingw64\bin
D:\mingw64\include
```

2，配置Path

![微信截图_20210506175025.png](/img/微信截图_20210506175025.png)

3，验证C/C++环境

按下win+r，打开cmd，输入 `gcc -v -E -x c++ -` 验证C

![微信截图_20210506175229.png](/img/微信截图_20210506175229.png)

C++环境验证：`g++ -v`

![微信截图_20210507100738.png](/img/微信截图_20210507100738.png)

## VSCode安装插件

安装 2 个扩展插件：C/C++扩展插件 和 Code Runner。

![微信截图_20210506165025.png](/img/微信截图_20210506165025.png)

![微信截图_20210507101147.png](/img/微信截图_20210507101147.png)

![微信截图_20210507101255.png](/img/微信截图_20210507101255.png)

## 验证配置

新建 `hello.c`，注意：路径中不能有中文

```c
#include <stdio.h>

int main()
{
   /* 我的第一个 C 程序 */
   printf("Hello, World! \n");
   
   return 0;
}
```

如果生成的 `.exe` 文件打开时会一闪而过，`return 0;` 前加入 `system("pause"); 语句。

```c
system("pause");  //暂停函数，请按任意键继续...
```

gcc 进行 c 语言编译分为四个步骤：

1.预处理，生成预编译文件（.i 文件）：

```c
gcc –E hello.c –o hello.i
```

2.编译，生成汇编代码（.s 文件）：

```c
gcc –S hello.i –o hello.s
```

3.汇编，生成目标文件（.o 文件）：

```c
gcc –c hello.s –o hello.o
```

4.链接，生成可执行文件：

```c
gcc hello.o –o hello
```

参考：

[VS Code C语言开发环境配置（保姆教程，详细到安装过程的每一步）](https://blog.csdn.net/incredibleimpact/article/details/109733494)

[VS Code运行C和C++程序](http://c.biancheng.net/view/8114.html)