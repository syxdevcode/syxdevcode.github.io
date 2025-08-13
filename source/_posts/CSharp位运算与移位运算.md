---
title: CSharp位运算与移位运算
date: 2019-11-15 15:54:17
tags:
- DotNet
- CSharp
- CSharp基础
categories: 
- CSharp基础
---

移位运算符仅针对 int、uint、long 和 ulong 类型定义，因此运算的结果始终包含至少 32 位。 如果左侧操作数是其他整数类型（sbyte、byte、short、ushort 或 char），则其值将转换为 int 类型。

## 左移位运算符 <<

`<<` 运算符将其左侧操作数向左移动右侧操作数定义的位数。
左移运算会放弃超出结果类型范围的高阶位，并将低阶空位位置设置为零

```cs
uint x = 0b_1100_1001_0000_0000_0000_0000_0001_0001;
Console.WriteLine($"Before: {Convert.ToString(x, toBase: 2)}");

uint y = x << 4;
Console.WriteLine($"After:  {Convert.ToString(y, toBase: 2)}");
// Output:
// Before: 11001001000000000000000000010001
// After:  10010000000000000000000100010000
```

总结：左移相当于 X * 2的N次方 (乘以 2 的N次方)

```cs
x<<1= x*2
x<<2= x*4
x<<3= x*8
x<<4= x*16
```

## 右移位运算符 >>

`>>` 运算符将其左侧操作数向右移动右侧操作数定义的位数。
右移位运算会放弃低阶位。

```cs
uint x = 0b_1001;
Console.WriteLine($"Before: {Convert.ToString(x, toBase: 2), 4}");

uint y = x >> 2;
Console.WriteLine($"After:  {Convert.ToString(y, toBase: 2), 4}");
// Output:
// Before: 1001
// After:    10
```

总结：左移相当于 X / 2的N次方 (除以 2 的N次方)

```cs
x>>1= x/2
x>>2= x/4
x>>3= x/8
x>>4=x/16
```

参考：

[位运算符和移位运算符（C# 参考）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/operators/bitwise-and-shift-operators#left-shift-operator-)

[C#移位运算(左移和右移)](https://www.cnblogs.com/kakimsun/archive/2010/04/27/1722403.html)