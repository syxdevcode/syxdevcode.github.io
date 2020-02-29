---
title: CSharp-认识反射发出
date: 2019-01-15 08:51:32
tags:
- DotNet
- CSharp
- Emit(反射发出)
categories: 
- Emit(反射发出)
---
参考：[发出动态方法和程序集](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/emitting-dynamic-methods-and-assemblies)

`System.Reflection.Emit` 命名空间提供的功能被称为反射发出。 `System.Reflection.Emit` 命名空间中的一组托管类型，它们允许编译器或工具在运行时发出元数据和 Microsoft 中间语言 (MSIL)，并在磁盘上生成可移植可执行 (PE) 文件（可选）。

反射发出具有以下功能：

* 在运行时定义轻量全局方法（使用 `DynamicMethod` 类）并通过委托执行这些方法。
* 在运行时定义程序集，然后运行程序集以及/或者将程序集保存到磁盘。
* 在运行时定义程序集，运行程序集，然后卸载程序集并允许垃圾回收回收其资源。
* 在运行时在新的程序集中定义模块，然后运行模块以及/或者将模块保存到磁盘。
* 在运行时在模块中定义类型，创建这些类型的实例并调用其方法。
* 为已定义模块定义可供工具（如调试器和代码探查器）使用的符号化信息。

参考：

[OpCodes](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit.opcodes?view=netframework-4.7.2)

[System.Reflection.Emit](https://docs.microsoft.com/zh-cn/dotnet/api/system.reflection.emit?view=netframework-4.7.2)