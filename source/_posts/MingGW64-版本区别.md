---
title: MingGW64 版本区别
date: 2021-05-06 20:21:26
tags:
- C语言
- VSCode
- MingGW64
categories: C语言
---

```
x86_64-posix-sjlj
x86_64-posix-seh
x86_64-win32-sjlj
x86_64-win32-seh
i686-posix-sjlj
i686-posix-dwarf
i686-win32-sjlj
i686-win32-dwarf
```

`POSIX`：表示可移植操作系统接口(Portable Operating System Interface of UNIX，缩写为 `POSIX` )

**解释**

DWARF：一种带调试信息(DWARF- 2（DW2）EH)的包, 所以比一般的包尺寸大，仅支持32位系统
SJLJ：跨平台，支持32，64位系统，缺点是：运行速度稍慢，GCC不支持
SEH: 调用系统机制处理异常，支持32，64位系统，缺点是：Gcc不支持（即将支持）

<!--more-->
**解释**

x86_64: 简称X64，64位操作系统
i686: 32位操作系统 (i386的子集)，差不多奔腾2(1997年5月)之后的CPU都是可以用的；

**解释**

posix: 启用了C++ 11 多线程特性
win32: 未启用 （从时间线上正在尝试也启用部分 Threading）

**区别**

DWARF DWARF- 2（DW2）EH ，这需要使用DWARF-2（或DWARF-3）调试信息。 DW-2 EH可以导致可执行文件略显膨胀，因为大的调用堆栈解开表必须包含在可执行文件中。
setjmp / longjmp（SJLJ）。基于SJLJ的EH比DW2 EH慢得多（在没有异常时会惩罚甚至正常执行），但是可以在没有使用GCC编译的代码或没有调用堆栈的代码上工作。
结构化异常处理（SEH） （Structured Exception Handling）Windows使用自己的异常处理机制。

[MingGW64 版本区别于各版本说明](https://www.pcyo.cn/linux/20181212/216.html)