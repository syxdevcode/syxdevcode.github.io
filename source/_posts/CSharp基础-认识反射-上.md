---
title: CSharp基础-认识反射-上
date: 2019-01-11 08:47:54
tags:
- DotNet
- CSharp
- 反射
- CSharp基础
categories: 
- 反射
---
# CSharp基础-认识反射-上

本章包括查看对象类型信息，反射类型和泛型反射，反射检查和实例化泛型类型。

## 反射概述

在程序中，当我们需要动态的去加载程序集的时候(将对程序集的引用由编译时推移到运行时)，反射是一种很好的选择。可以使用反射动态地创建类型的实例，将类型绑定到现有对象，或从现有对象中获取类型，然后调用其方法或访问其字段和属性。反射为.NET类型提供了高度的动态能力，包括：元数据的动态查询、绑定与执行、动态代码生成。常用的反射类型包含在`System.Reflection` 和 `System.Reflection.Emit` ,反射包括程序集反射、类型反射、接口反射、类型成员反射。

反射在以下情况下很有用：

* 需要访问程序元数据中的特性时。
* 检查和实例化程序集中的类型。
* 在运行时构建新类型。 使用 `System.Reflection.Emit` 中的类。
* 执行后期绑定，访问在运行时创建的类型上的方法。 请参阅主题 Dynamically Loading and Using Types（动态加载和使用类型）。

## 运行时类型标识(RTTI)

运行时类型信息 (RTTI,Run-Time Type Information) 是一种允许在程序执行过程中确定对象类型的机制。例如使用它能够确切地知道基类引用指向了什么类型对象。运行时类型标识，能预先测试某个强制类型转换操作，能否成功，从而避免无效的强制类型转换异常。

在c#中有三个支持RTTI的关键字：`is` 、 `as`  、`typeof`。

### is 运算符

参考：[is（C# 参考）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/is)

能够判断对象类型是否为特定类型，如果两种类型是相同类型，或者两者之间存在引用，装箱拆箱转换，则表明两种类型是兼容的。或（从 C# 7.0 开始）针对某个模式测试表达式。

#### 类型兼容性测试

语法：

```cs
expr is type

if (obj is Person) {
   // Do something if obj is a Person.
}

double number1 = 12.63;
if (Math.Ceiling(number1) is double)
Console.WriteLine("The expression returns a double.");
```

expr 是计算结果为某个类型的实例的表达式，而 type 是 expr 结果要转换到的类型的名称。
expr 可以是返回值的任何表达式，匿名方法和 Lambda 表达式除外。

如果满足以下条件，则 `is` 语句为 true：

* expr 是与 type 具有相同类型的一个实例。
* expr 是派生自 type 的类型的一个实例。 换言之，expr 结果可以向上转换为 type 的一个实例。
* expr 具有属于 type 的一个基类的编译时类型，expr 还具有属于 type 或派生自 type 的运行时类型。 变量的编译时类型是其声明中定义的变量类型。 变量的运行时类型是分配给该变量的实例类型。
* expr 是实现 type 接口的类型的一个实例。

#### 利用 is 的模式匹配

从 C# 7.0 开始，`is` 和 [switch](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/switch) 语句支持模式匹配。

** 类型模式**

用于测试表达式是否可转换为指定类型，如果可以，则将其转换为该类型的一个变量。

语法：

```cs
expr is type varname
```

如果 expr 为 true，且 `is` 与 `if` 语句一起使用，则会分配 varname，并且其仅在 `if` 语句中具有局部范围。

```cs
if (o is Employee e)
{
    return Name.CompareTo(e.Name);
}
```

** 常量模式**

使用常量模式执行模式匹配时，`is` 会测试表达式结果是否等于指定常量。

```cs
expr is constant
```

其中 expr 是要计算的表达式，constant 是要测试的值。 constant 可以是以下任何常数表达式：

* 一个文本值。
* 已声明 const 变量的名称。
* 一个枚举常量。

常数表达式的计算方式如下：

* 如果 expr 和 constant 均为整型类型，则 C# 相等运算符确定表示式是否返回 true（即，是否为 `expr == constant`）。
* 否则，由对静态 `Object.Equals(expr, constant)` 方法的调用来确定表达式的值。

`is` 语句支持 null 关键字。

```cs
expr is null

if (o is null)
{}
```

** var 模式**

具有 var 模式的模式匹配始终成功。 语法为

```cs
expr is var varname
```

```cs
foreach (var item in items) {
    if (item is var obj)
    Console.WriteLine($"Type: {obj.GetType().Name}, Value: {obj}");
}
```

请注意，如果 expr 为 null，则 is 表达式仍为 true 并向 varname 分配 null。

### as 运算符

参考：[as（C# 参考）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/as)

在运行期间执行类型转换，并且能够使得类型转换失败不抛异常，而返回一个null值。

```cs
expression as type  
```

```cs
Base b = d as Base;
if (b != null)
{
    Console.WriteLine(b.ToString());
}
```

### typeof运算符

参考：[typeof（C# 参考）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/typeof)

获取类型的System.Type 对象。

```cs
System.Type type = typeof(int);
```

获取表达式的运行时类型:

```cs
int i = 0;
System.Type type = i.GetType();
```

示例显示如何确定方法的返回类型是否为泛型 `IEnumerable<T>`。 如果返回类型不是 `IEnumerable<T>` 泛型类型，则 `Type.GetInterface` 将返回 `null`。

```cs
MethodInfo method = typeof(string).GetMethod("Copy");
Type t = method.ReturnType.GetInterface(typeof(IEnumerable<>).Name);
if (t != null)
    Console.WriteLine(t);
else
    Console.WriteLine("The return type is not IEnumerable<T>.");

Console.ReadKey();
```

## 查看类型信息

参考：[查看类型信息](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/viewing-type-information)

### 模块和程序集

模块是程序集中代码的逻辑集合。您可以在程序集内部拥有多个模块，并且每个模块都可以使用不同的.NET语言编写（VS不支持创建多模块程序集）。

<font color=#ff0000 size=4 face="黑体">程序集包含模块。模块包含类。类包含函数。</font>

`System.Type` 类是反射的中心。 当反射提出请求时，公共语言运行时为已加载的类型创建 Type。 可使用 Type 对象的方法、字段、属性和嵌套类来查找该类型的任何信息。

`Type` 类派生于 `System.Reflection.MemberInfo` 抽象类。

获取程序集的 `Assembly` 对象和模块:

```cs
Assembly a = typeof(object).Module.Assembly;
```

### Type.GetTypes()

从已加载的程序集获取 Type 对象:

```cs
// Loads an assembly using its file name.
Assembly a = Assembly.LoadFrom("MyExe.exe");
// Gets the type names from the assembly.
Type[] types2 = a.GetTypes();
foreach (Type t in types2)
{
    Console.WriteLine(t.FullName);
}
```

使用 `Type.Module` 属性获取一个封装含该类型的模块的对象。 使用 `Module.Assembly` 属性查找一个封装含该模块的程序集的对象。 可以获取直接使用 `Type.Assembly` 属性封装类型的程序集。

### Type.GetMembers()

至少要有Instance（或Static）与Public（或NonPublic）标记。

```cs
Type.GetMembers() // 获取所有公共成员。不包括 private 和 protected 访问权限
Type.GetMembers(BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.Public | BindingFlags.DeclaredOnly | BindingFlags.Static) // 包含非公开;包含实例成员;包含公开;只包含本类声明的成员，不考虑继承的成员;包含静态成员。
```

```cs
Type.GetFields() // 获取字段
Type.GetProperties() // 获取属性
Type.GetEvents() // 获取事件
Type.GetMethods() // 获取方法
```

### Type.ConstructorInfo()

列出类的构造函数

```cs
private static void Main(string[] args)
{
    Type t = typeof(System.String);
    Console.WriteLine("Listing all the public constructors of the {0} type", t);
    // Constructors.
    ConstructorInfo[] ci = t.GetConstructors(BindingFlags.Public | BindingFlags.Instance);
    Console.WriteLine("//Constructors");
    PrintMembers(ci);

    Console.ReadKey();
}

public static void PrintMembers(MemberInfo[] ms)
{
    foreach (MemberInfo m in ms)
    {
        Console.WriteLine("{0}{1}", "     ", m);
    }
    Console.WriteLine();
}
```

## 反射类型和泛型类型

从反射的角度来说，泛型类型和普通类型之间的区别在于泛型类型具有与之关联的一组类型形参（若是泛型类型定义）或类型实参（若是构造类型）。

### 泛型类型和泛型方法

检查未知类型（由 Type的实例表示），使用 `IsGenericType` 属性来确定未知类型是否为泛型。如果类型是泛型，它会返回 true 。
检查未知方法（由 MethodInfo 类的实例表示）时，请使用 `IsGenericMethod` 属性来确定此方法是否为泛型。

### 泛型类型定义和泛型方法定义

使用 `IsGenericTypeDefinition` 属性来确定 `Type` 对象是否表示泛型类型定义，并使用 `IsGenericMethodDefinition` 方法来确定 `MethodInfo` 是否表示泛型方法定义。

详细请参考：[Type.IsGenericTypeDefinition Property](https://docs.microsoft.com/zh-cn/dotnet/api/system.type.isgenerictypedefinition?view=netframework-4.7.2)

[Type.GetGenericTypeDefinition Method](https://docs.microsoft.com/zh-cn/dotnet/api/system.type.getgenerictypedefinition?view=netframework-4.7.2)

### 类型或方法是开放式还是封闭式

* 开放类型：具有泛型类型参数的类型
* 封闭类型：为所有类型参数都传递了实际的数据类型

如果类型为开放式， `Type.ContainsGenericParameters` 属性将返回 true 。 对于方法， `MethodBase.ContainsGenericParameters` 方法执行相同的功能。

### 生成封闭式泛型类型

`MakeGenericType` 方法来创建封闭式泛型类型;
`MakeGenericMethod` 方法来为封闭式泛型方法创建 `MethodInfo`;

开放式泛型类型或方法（非泛型类型或方法定义），不能创建它的实例，也不能提供缺少的类型形参。

`GetGenericTypeDefinition` 方法来获取泛型类型定义;
`GetGenericMethodDefinition` 方法来获取泛型方法定义。

例如，如果你有一个表示 `Type` 的 `Dictionary<int, string>` 对象且想创建类型 `Dictionary<string, MyClass>`，则可以使用 `GetGenericTypeDefinition` 方法来获取表示 `Type` 的 `Dictionary<TKey, TValue>` ，然后使用 `MakeGenericType` 方法来生成表示 `Type` 的 `Dictionary<int, MyClass>`。

### 检查类型实参和类型形参

`Type.GetGenericArguments` 方法来获取 `Type` 对象的数组（表示泛型类型的类型形参或类型实参）; `MethodInfo.GetGenericArguments` 方法为泛型方法获取 `MethodInfo`（表示泛型类型的类型形参或类型实参）。如果元素是类型形参，则 `IsGenericParameter` 属性为 `true` 。

## 如何使用反射

参考：[如何：使用反射检查和实例化泛型类型](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/how-to-examine-and-instantiate-generic-types-with-reflection)

获取泛型类型信息的方式与获取其他类型信息的方式相同：检查表示泛型类型的 Type 对象。 主要的差异在于，泛型类型具有表示其泛型类型参数的 Type 对象列表。

### 检查泛型类型和类型参数

范例：

```cs
private static void Main(string[] args)
{
    // 1,获取表示泛型类型的 Type 实例
    Type t = typeof(Dictionary<,>);

    // 2, 使用 IsGenericType 属性确定类型是否为泛型，
    // 然后使用 IsGenericTypeDefinition 属性确定类型是否为泛型类型定义。
    Console.WriteLine("   Is this a generic type? {0}",
        t.IsGenericType);
    Console.WriteLine("   Is this a generic type definition? {0}",
        t.IsGenericTypeDefinition);

    // 3,使用 GetGenericArguments 方法获取包含泛型类型参数的数组。
    Type[] typeParameters = t.GetGenericArguments();

    // 4,使用 IsGenericParameter 属性确定它是类型形参（例如，在泛型类型定义中），
    // 还是为类型形参指定的类型（例如，在构造类型中）
    Console.WriteLine("   List {0} type arguments:",
        typeParameters.Length);
    foreach (Type tParam in typeParameters)
    {
        if (tParam.IsGenericParameter)
        {
            DisplayGenericParameter(tParam);
        }
        else
        {
            Console.WriteLine("      Type argument: {0}",
                tParam);
        }
    }
    Console.ReadKey();
}

private static void DisplayGenericParameter(Type tp)
{
    // 5,表示泛型类型参数的 Type 对象的名称和参数位置。
    Console.WriteLine("      Type parameter: {0} position {1}",
        tp.Name, tp.GenericParameterPosition);

    Type classConstraint = null;

    // 6,GetGenericParameterConstraints 方法获取单个数组中的所有约束，
    // 确定泛型类型参数的基类型约束和接口约束。无序
    foreach (Type iConstraint in tp.GetGenericParameterConstraints())
    {
        if (iConstraint.IsInterface)
        {
            Console.WriteLine("         Interface constraint: {0}",
                iConstraint);
        }
    }

    if (classConstraint != null)
    {
        Console.WriteLine("         Base type constraint: {0}",
            tp.BaseType);
    }
    else
        Console.WriteLine("         Base type constraint: None");

    // 7, GenericParameterAttributes 属性发现类型参数的特殊约束
    GenericParameterAttributes sConstraints =
        tp.GenericParameterAttributes &
        GenericParameterAttributes.SpecialConstraintMask;

    if (sConstraints == GenericParameterAttributes.None)
    {
        Console.WriteLine("         No special constraints.");
    }
    else
    {
        if (GenericParameterAttributes.None != (sConstraints &
                                                GenericParameterAttributes.DefaultConstructorConstraint))
        {
            Console.WriteLine("         Must have a parameterless constructor.");
        }
        if (GenericParameterAttributes.None != (sConstraints &
                                                GenericParameterAttributes.ReferenceTypeConstraint))
        {
            Console.WriteLine("         Must be a reference type.");
        }
        if (GenericParameterAttributes.None != (sConstraints &
                                                GenericParameterAttributes.NotNullableValueTypeConstraint))
        {
            Console.WriteLine("         Must be a non-nullable value type.");
        }
    }
}
```

### 构造泛型类型的实例

需要指定其泛型类型参数的实际类型，否则不能创建泛型类型的实例。使用 `MakeGenericType`方法。

实例：

```cs
using System;
using System.Collections.Generic;
using System.Reflection;
using System.Security.Permissions;

namespace ConsoleApp1
{
    // Define an example interface.
    public interface ITestArgument { }

    // Define an example base class.
    public class TestBase { }

    // Define a generic class with one parameter. The parameter
    // has three constraints: It must inherit TestBase, it must
    // implement ITestArgument, and it must have a parameterless
    // constructor.
    public class Test<T> where T : TestBase, ITestArgument, new() { }

    // Define a class that meets the constraints on the type
    // parameter of class Test.
    public class TestArgument : TestBase, ITestArgument
    {
        public TestArgument()
        {
        }
    }

    public class Example
    {
        // The following method displays information about a generic
        // type.
        private static void DisplayGenericType(Type t)
        {
            Console.WriteLine("\r\n {0}", t);
            Console.WriteLine("   Is this a generic type? {0}",
                t.IsGenericType);
            Console.WriteLine("   Is this a generic type definition? {0}",
                t.IsGenericTypeDefinition);

            // Get the generic type parameters or type arguments.
            Type[] typeParameters = t.GetGenericArguments();

            Console.WriteLine("   List {0} type arguments:",
                typeParameters.Length);
            foreach (Type tParam in typeParameters)
            {
                if (tParam.IsGenericParameter)
                {
                    DisplayGenericParameter(tParam);
                }
                else
                {
                    Console.WriteLine("      Type argument: {0}",
                        tParam);
                }
            }
        }

        // The following method displays information about a generic
        // type parameter. Generic type parameters are represented by
        // instances of System.Type, just like ordinary types.
        private static void DisplayGenericParameter(Type tp)
        {
            Console.WriteLine("      Type parameter: {0} position {1}",
                tp.Name, tp.GenericParameterPosition);

            Type classConstraint = null;

            foreach (Type iConstraint in tp.GetGenericParameterConstraints())
            {
                if (iConstraint.IsInterface)
                {
                    Console.WriteLine("         Interface constraint: {0}",
                        iConstraint);
                }
            }

            if (classConstraint != null)
            {
                Console.WriteLine("         Base type constraint: {0}",
                    tp.BaseType);
            }
            else
                Console.WriteLine("         Base type constraint: None");

            GenericParameterAttributes sConstraints =
                tp.GenericParameterAttributes &
                GenericParameterAttributes.SpecialConstraintMask;

            if (sConstraints == GenericParameterAttributes.None)
            {
                Console.WriteLine("         No special constraints.");
            }
            else
            {
                if (GenericParameterAttributes.None != (sConstraints &
                    GenericParameterAttributes.DefaultConstructorConstraint))
                {
                    Console.WriteLine("         Must have a parameterless constructor.");
                }
                if (GenericParameterAttributes.None != (sConstraints &
                    GenericParameterAttributes.ReferenceTypeConstraint))
                {
                    Console.WriteLine("         Must be a reference type.");
                }
                if (GenericParameterAttributes.None != (sConstraints &
                    GenericParameterAttributes.NotNullableValueTypeConstraint))
                {
                    Console.WriteLine("         Must be a non-nullable value type.");
                }
            }
        }

        [PermissionSet(SecurityAction.Demand, Name = "FullTrust")]
        public static void Main()
        {
            // Two ways to get a Type object that represents the generic
            // type definition of the Dictionary class.
            //
            // Use the typeof operator to create the generic type
            // definition directly. To specify the generic type definition,
            // omit the type arguments but retain the comma that separates
            // them.
            Type d1 = typeof(Dictionary<,>);

            // You can also obtain the generic type definition from a
            // constructed class. In this case, the constructed class
            // is a dictionary of Example objects, with String keys.
            Dictionary<string, Example> d2 = new Dictionary<string, Example>();
            // Get a Type object that represents the constructed type,
            // and from that get the generic type definition. The
            // variables d1 and d4 contain the same type.
            Type d3 = d2.GetType();
            Type d4 = d3.GetGenericTypeDefinition();

            // Display information for the generic type definition, and
            // for the constructed type Dictionary<String, Example>.
            DisplayGenericType(d1);
            DisplayGenericType(d2.GetType());

            // Construct an array of type arguments to substitute for
            // the type parameters of the generic Dictionary class.
            // The array must contain the correct number of types, in
            // the same order that they appear in the type parameter
            // list of Dictionary. The key (first type parameter)
            // is of type string, and the type to be contained in the
            // dictionary is Example.
            Type[] typeArgs = { typeof(string), typeof(Example) };

            // Construct the type Dictionary<String, Example>.
            Type constructed = d1.MakeGenericType(typeArgs);

            DisplayGenericType(constructed);

            object o = Activator.CreateInstance(constructed);

            Console.WriteLine("\r\nCompare types obtained by different methods:");
            Console.WriteLine("   Are the constructed types equal? {0}",
                (d2.GetType() == constructed));
            Console.WriteLine("   Are the generic definitions equal? {0}",
                (d1 == constructed.GetGenericTypeDefinition()));

            // Demonstrate the DisplayGenericType and
            // DisplayGenericParameter methods with the Test class
            // defined above. This shows base, interface, and special
            // constraints.
            DisplayGenericType(typeof(Test<>));

            Console.ReadKey();
        }
    }
}
```

参考：

[反射 (C#)](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/reflection)

[.NET Framework 中的反射](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/reflection)

[C#之玩转反射](http://www.cnblogs.com/yaozhenfa/p/CSharp_Reflection_1.html)

[C#反射用法，入门到精通](http://www.codeisbug.com/Ask/11/1041)