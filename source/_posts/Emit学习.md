---
title: Emit学习
date: 2017-03-02 11:00:30
tags: 
- 反射
- CSharp基础
- Emit(反射发出)
categories: 
- Emit(反射发出)
---
## 动态创建完整的程序集

### 创建程序集

``` csharp
using System;
using System.Reflection;
using System.Reflection.Emit;

namespace EmitTest
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            AssemblyName assemblyName = new AssemblyName("EmitTest.MvcAdviceProvider");

            AssemblyBuilder assemblyBuilder = AppDomain.CurrentDomain.DefineDynamicAssembly(assemblyName, AssemblyBuilderAccess.Run);
        }
    }
}
```

`AppDomain.CurrentDomain.DefineDynamicAssembly`方法返回一个AssemblyBuilder实例。

其中，第一个参数是AssemblyName实例，是程序集的唯一标识；第二个参数AssemblyBuilderAccess.Run表明该程序集只能用来执行代码，

不能被持久保存。AssemblyBuilderAccess还有如下选项：

  `AssemblyBuilderAccess.ReflectionOnly`：程序集只能在反射上下文中执行。

  `AssemblyBuilderAccess.RunAndCollect`：程序集可以运行和垃圾回收。

  `AssemblyBuilderAccess.RunAndSave`：程序集可以执行代码而且被持久保存。

  `AssemblyBuilderAccess.Save`：程序集是持久化的，保存之前不可以执行代码。

创建了程序集之后，我们继续向程序集中添加模块。

#### 向程序集添加模块

``` csharp
using System;
using System.Reflection;
using System.Reflection.Emit;

namespace EmitTest
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            AssemblyName assemblyName = new AssemblyName("EmitTest.MvcAdviceProvider");

            AssemblyBuilder assemblyBuilder = AppDomain.CurrentDomain.DefineDynamicAssembly(assemblyName, AssemblyBuilderAccess.Run);

            ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("MvcAdviceProvider");
        }
    }
}
```

`AssemblyBuilder.DefineDynamicModule`方法来创建模块，该方法共有三个重载，如下表所示：

`DefineDynamicModule(String)`定义指定名称的模块。

`DefineDynamicModule(String, Boolean)`定义指定名称的模块，并指定是否发出符号信息。

`DefineDynamicModule(String, String)`定义持久模块。用给定名称定义将保存到指定文件路径的模块。不发出符号信息。

`DefineDynamicModule(String, String, Boolean)`定义持久模块，并指定模块名称、用于保存模块的文件名，同时指定是否使用默认符号编写器发出符号信息。

#### 定义类型

我们要定义的类型必须继承并实现IAssessmentAopAdviceProvider接口

```csharp
TypeBuilder typeBuilder = moduleBuilder.DefineType("EmitTest.MvcAdviceProvider", TypeAttributes.Public,
                                    typeof(object), new Type[] { typeof(IAssessmentAopAdviceProvider) });
```

上述代码中mb.DefineType方法返回一个TypeBuilder实例，该方法有6个重载方法，这里采用的方法有四个参数，

第一个参数是类型名称，第二个参数的TypeAttributes枚举是类型的访问级别和类型类别等其他信息，第三个参数

是类型继承的基类，第四个参数是类型实现的接口。其他重载函数的说明如下（引自MSDN）：

`DefineType(String)`在此模块中用指定的名称为私有类型构造 TypeBuilder。

`DefineType(String, TypeAttributes)`在给定类型名称和类型特性的情况下，构造 TypeBuilder。

`DefineType(String, TypeAttributes, Type)`在给定类型名称、类型特性和已定义类型扩展的类型的情况下，构造 TypeBuilder。

`DefineType(String, TypeAttributes, Type, Int32)`在给定类型名称、特性、已定义类型扩展的类型和类型的总大小的情况下，构造 TypeBuilder。

`DefineType(String, TypeAttributes, Type, PackingSize)`在给定类型名称、特性、已定义类型扩展的类型和类型的封装大小的情况下，构造 TypeBuilder。

`DefineType(String, TypeAttributes, Type, Type[])`在给定类型名称、特性、已定义类型扩展的类型和已定义类型实现的接口的情况下，构造 TypeBuilder。

`DefineType(String, TypeAttributes, Type, PackingSize, Int32)`在给定类型名称、特性、已定义类型扩展的类型，已定义类型的封装

通过TypeBuilder，可以使用TypeBuilder.DefineField来定义字段，使用TypeBuilder.DefineConstructor来定义构造函数，

使用TypeBuilder.DefineMethod来定义方法，并使用TypeBuilder.DefineEvent来定义事件等，总之可以定义类型里的任何成员。

这里我们只需要定义方法

```
MethodBuilder beforeMethodBuilder = typeBuilder.DefineMethod("Before", MethodAttributes.Public, typeof(object), new Type[] { typeof(object) });

MethodBuilder afterMethodBuilder = typeBuilder.DefineMethod("After", MethodAttributes.Public, typeof(object), new Type[] { typeof(object), typeof(object) });
```

在上面的代码中，使用TypeBuilder.DefineMethod 方法来创建MethodBuilder对象。该方法有5个重载，如下表（引自MSDN）:

`DefineMethod(String, MethodAttributes)`使用指定的名称和方法特性向类型中添加新方法。

`DefineMethod(String, MethodAttributes, CallingConventions)`使用指定名称、方法特性和调用约定向类型中添加新方法。

`DefineMethod(String, MethodAttributes, Type, Type[])`使用指定的名称、方法特性和调用约定向类型中添加新方法。

`DefineMethod(String, MethodAttributes, CallingConventions, Type, Type[])`使用指定的名称、方法特性、调用约定和方法签名向类型中添加新方法。

`DefineMethod(String, MethodAttributes, CallingConventions, Type, Type[], Type[], Type[], Type[][], Type[][])`使用指定的名称、方法特性、调用约定、方法签名和自定义修饰符向类型中添加新方法。

完整代码：

```
using System;
using System.Reflection;
using System.Reflection.Emit;

namespace EmitTest
{
    internal class Program
    {
        private static void Main(string[] args)
        {
            AssemblyName assemblyName = new AssemblyName("EmitTest.MvcAdviceProvider");

            AssemblyBuilder assemblyBuilder = AppDomain.CurrentDomain.DefineDynamicAssembly(assemblyName, AssemblyBuilderAccess.Run);

            ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("MvcAdviceProvider");

            TypeBuilder typeBuilder = moduleBuilder.DefineType("EmitTest.MvcAdviceProvider", TypeAttributes.Public,

                                                               typeof(object), new Type[] { typeof(IAssessmentAopAdviceProvider) });

            MethodBuilder beforeMethodBuilder = typeBuilder.DefineMethod("Before", MethodAttributes.Public, typeof(object), new Type[] { typeof(object) });

            MethodBuilder afterMethodBuilder = typeBuilder.DefineMethod("After", MethodAttributes.Public, typeof(object), new Type[] { typeof(object), typeof(object) });

            TestType(typeBuilder);
        }

        private static void TestType(TypeBuilder typeBuilder)
        {
            Console.WriteLine(typeBuilder.Assembly.FullName);

            Console.WriteLine(typeBuilder.Module.Name);

            Console.WriteLine(typeBuilder.Namespace);

            Console.WriteLine(typeBuilder.Name);

            Console.Read();
        }
    }
}
```

参考：

[http://www.cnblogs.com/xuanhun/archive/2012/06/03/2532922.html](http://www.cnblogs.com/xuanhun/archive/2012/06/03/2532922.html "Emit基本操作")
