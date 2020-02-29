---
title: CSharp-使用程序集编程
date: 2019-01-24 16:15:02
tags:
- CSharp
- 程序集
categories: 
- 程序集
---
# 使用程序集编程

程序集是 .NET Framework 的构造块；它们构成了部署、版本控制、重复使用、激活范围和安全权限的基本单元。 程序集向公共语言运行时提供了解类型实现所需要的信息。 程序集是为协同工作而生成的类型和资源的集合，这些类型和资源构成了一个逻辑功能单元。 对于运行时，类型不存在于程序集上下文之外。

## 程序集名称

程序集的名称存储在元数据中，它对程序集的范围及应用程序对程序集的使用有重要影响。 <font color=#ff0000 size=4 face="黑体">强名称程序集有一个完全限定的名称，由程序集的名称、区域性、公钥及版本号组成。 该名称通常称为显示名称，对于加载的程序集，可通过使用 FullName 属性来获取它。</font>

在 .NET Framework 2.0 版中，向程序集标识添加了处理器体系结构，从而允许使用特定于处理器的程序集版本。

例如：

完全限定名表明 myTypes 程序集的强名称具有公钥标记、区域性值为美国英语、版本号为 1.0.1234.0。 它的处理器体系结构为“msil”，表示程序集将以实时 (JIT) 方式编译为 32 位代码或 64 位代码（具体取决于操作系统和处理器）。

```cs
myTypes, Version=1.0.1234.0, Culture=en-US, PublicKeyToken=b77a5c561934e089c, ProcessorArchitecture=msil  
```

<font color=#ff0000 size=4 face="黑体">请求程序集中的类型的代码必须使用完全限定的程序集名称。 这称为完全限定绑定。</font> 在 .NET Framework 中引用程序集时不允许使用部分绑定，因为它只指定一个程序集名称。

对组成 .NET Framework 的程序集的所有程序集引用也必须包含程序集的完全限定名。对于 .NET Framework 程序集，区域性值始终不是特定的。

```cs
System.data, version=1.0.3300.0, Culture=neutral, PublicKeyToken=b77a5c561934e089  
```

绑定到程序集时，运行时不区分程序集名称的大小写，但会保留程序集名称中使用的大小写。

<font color=#ff0000 size=4 face="黑体">如果将强名称程序集置于全局程序集缓存中，则程序集的文件名必须与程序集名称相匹配（不包括文件扩展名，如 .exe 或 .dll）。</font> 例如，如果程序集的文件名为 myAssembly.dll，则程序集名称必须为 myAssembly。 <font color=#ff0000 size=4 face="黑体">只有在根应用程序目录中部署的专用程序集的程序集名称可以不同于文件名。</font>

## 确定程序集完全限定的名称

1,在全局程序集缓存中查找一个程序集的完全限定名，请使用全局程序集缓存工具 ([Gacutil.exe](https://docs.microsoft.com/zh-cn/dotnet/framework/tools/gacutil-exe-gac-tool))。

2,不在全局程序集缓存中的程序集，可以通过多种方式获取完全限定的程序集名称：可使用代码将信息输出到控制台或变量，也可使用 [Ildasm.exe（IL 反汇编程序）](https://docs.microsoft.com/zh-cn/dotnet/framework/tools/ildasm-exe-il-disassembler)检查包含完全限定名的程序集元数据。

如果应用程序已加载程序集，则可检索 Assembly.FullName 属性的值在以获取完全限定名。

```cs
Type t = typeof(System.Data.DataSet);
string s = t.Assembly.FullName.ToString();
```

如果知道程序集的文件系统路径，则可调用静态AssemblyName.GetAssemblyName 方法获取完全限定的程序集名称。

```cs
Console.WriteLine(AssemblyName.GetAssemblyName(@".\UtilityLibrary.dll"));
```

## 设置程序集特性

程序集特性是提供程序集相关信息的值。 特性分为以下几组信息：

* 程序集标识特性。
* 信息性特性。
* 程序集清单特性。
* 强名称特性。

参考：[设置程序集特性](https://docs.microsoft.com/zh-cn/dotnet/framework/app-domains/set-assembly-attributes)

[重温.NET下Assembly的加载过程](http://www.cnblogs.com/daxnet/p/8525249.html)