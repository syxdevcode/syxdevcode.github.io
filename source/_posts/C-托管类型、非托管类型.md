---
title: C#托管类型、非托管类型
date: 2020-10-16 17:47:44
tags:
- DotNet
- CSharp
- CSharp基础
- DotNet面试题解析
categories: 
- CSharp基础
---

## 托管类型

托管类型包括 引用类型 以及 包含有引用类型或托管类型成员的结构。

* 引用类型
* 含引用类型或托管类型成员（字段、自动实现 get 访问器的属性）的结构（managed structure）

## 非托管类型

如果某个类型是以下类型之一，则它是非托管类型 ：

* sbyte、byte、short、ushort、int、uint、long、ulong、char、float、double、decimal 或 bool
* 任何枚举类型
* 任何指针类型
* 任何用户定义的 struct 类型，只包含非托管类型的字段，并且在 C# 7.3 及更早版本中，不是构造类型（包含至少一个类型参数的类型）

参考：

[非托管类型（C# 参考）
](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/builtin-types/unmanaged-types)