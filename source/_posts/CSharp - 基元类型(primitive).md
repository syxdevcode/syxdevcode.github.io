---
title: CSharp - 基元类型(primitive)
date: 2019-07-02 18:15:01
tags:
- DotNet
- CSharp
- CSharp基础
categories: 
- CSharp基础
---
# CSharp - 基元类型(primitive)

## 基元类型

CLR 定义的基元（原始）类型：

`Boolean`, `Byte`, `SByte`, `Int16`, `UInt16`, `Int32`, `UInt32`, `Int64`, `UInt64`, `Char`, `Double`, `Single`。

<font color=#ff0000 size=4 face="黑体">注意：string不是基元（原始）类型。</font>

即：

```cs
System.Boolean
System.Byte
System.SByte
System.Int16
System.UInt16
System.Int32
System.UInt32
System.Int64
System.UInt64
System.IntPtr
System.UIntPtr
System.Char
System.Double
System.Single
```

通过 `System.Type` 提供 `IsPrimitive` 属性，指示当前类型是否为基元类型；

## 内置类型

<font color=#ff0000 size=4 face="黑体">C#内置类型和.NET框架中的类型有直接的映射关系。C#内置类型直接映射到Framework(FCL)中存在的类型。</font>如: 在用内置类型int初始化一个整数时，int会直接映射到FCL中 `System.Int32` 类型，这个过程,编译器自动完成。C#中的string是内置类型，直接映射到.NET框架中的String，所以没有任何不同。 

下图显示，C#中的所有内置类型和FCL类型：

![QQ截图20190705175059.png](/img/QQ截图20190705175059.png)

隐式转化：如果转化过程时安全的，也就是说在转化过程中数据不会丢失，那么久进行隐式转换。 
显示转换：转换过程不安全，意味着可能会丢失精度或者数量级，那就要求显示转换。

参考：

[C# 基元类型（primitive）](https://blog.csdn.net/kone0611/article/details/84856702)

[C#中基元类型、引用类型与值类型](https://blog.csdn.net/u014677855/article/details/82111358)

[内置类型表（C# 参考）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/built-in-types-table)

[Is String a primitive type?](https://stackoverflow.com/questions/3965752/is-string-a-primitive-type)

[Type.IsPrimitiveImpl Method](https://docs.microsoft.com/en-us/dotnet/api/system.type.isprimitiveimpl?redirectedfrom=MSDN&view=netframework-4.8#System_Type_IsPrimitiveImpl)