---
title: CSharp基础-泛型
date: 2018-12-26 09:56:02
tags:
- DotNet
- CSharp
- 集合与泛型
- CSharp基础
categories: 
- 集合与泛型
---

&emsp;&emsp;C# 语言和公共语言运行时 (CLR) 的 2.0 版本中添加了泛型。 泛型将类型参数的概念引入 .NET Framework，这样就可以设计具有以下特征的类和方法：<font color=##ff0000 size=4 face="黑体">在客户端代码声明并初始化这些类和方法之前，这些类和方法会延迟指定一个或多个类型。</font> 例如，通过使用泛型类型参数 T，可以编写其他客户端代码能够使用的单个类，而不会产生运行时转换或装箱操作的成本或风险。

<font color=##ff0000 size=4 face="黑体">泛型：为所存储或使用的一个或多个类型具有占位符（类型形参）的类、结构、接口和方法。</font> 泛型集合类可以将类型形参用作其存储的对象类型的占位符；类型形参呈现为其字段的类型和其方法的参数类型。 泛型方法可将其类型形参用作其返回值的类型或用作其形参之一的类型。

&emsp;&emsp;泛型类和泛型方法兼具可重用性、类型安全性和效率，这是非泛型类和非泛型方法无法实现的。 <font color=##ff0000 size=4 face="黑体">泛型通常与集合以及作用于集合的方法一起使用。通过创建泛型类，可在编译时创建类型安全的集合。</font>

## 泛型概述（优点）

* 使用泛型类型可以最大限度地重用代码、保护类型安全性以及提高性能。
* 避免了装箱和拆箱操作。
* 编译时类型检查。
* 泛型最常见的用途是创建集合类。
* .NET Framework 类库在 `System.Collections.Generic` 命名空间中包含几个新的泛型集合类。 应尽可能使用这些类来代替某些类，如 System.Collections 命名空间中的 ArrayList。
* 可以创建自己的泛型接口、泛型类、泛型方法、泛型事件和泛型委托。
* 可以对泛型类进行约束以访问特定数据类型的方法。
* 在泛型数据类型中所用类型的信息可在运行时通过使用反射来获取。

## 泛型类型参数

参考：[.NET 中的泛型](https://docs.microsoft.com/zh-cn/dotnet/standard/generics/index)

从反射的角度来说，泛型类型和普通类型之间的区别在于泛型类型具有与之关联的一组类型形参（若是泛型类型定义）或类型实参（若是构造类型）。

例如：

```cs
GenericList<T>; // T 类型形参
// 若要使用 GenericList<T>，客户端代码必须通过指定尖括号内的类型参数来声明并实例化构造类型(构造泛型类型)。
GenericList<float> list1 = new GenericList<float>();  // float 类型实参
```

## 类型参数的约束

&emsp;&emsp;约束告知编译器类型参数必须具备的功能。 在没有任何约束的情况下，类型参数可以是任何类型。 编译器只能假定 Object 的成员，它是任何 .NET 类型的最终基类。 如果客户端代码尝试使用约束所不允许的类型来实例化类，则会产生编译时错误。 <font color=#0099ff size=4 face="黑体">通过使用 where 上下文关键字指定约束。</font>

下表列出了七种类型的约束：

约束 | 说明
---- | ---
where T : struct | 类型参数必须是值类型。 可以指定除 Nullable<T> 以外的任何值类型。
where T : class  | 类型参数必须是引用类型。 此约束还应用于任何类、接口、委托或数组类型。
where T : unmanaged | 类型参数不能是引用类型，并且任何嵌套级别均不能包含任何引用类型成员。
where T : new() | 类型参数必须具有公共无参数构造函数。 与其他约束一起使用时，new() 约束必须最后指定。
where T : <基类名> | 类型参数必须是指定的基类或派生自指定的基类。
where T : <接口名称> | 类型参数必须是指定的接口或实现指定的接口。 可指定多个接口约束。 约束接口也可以是泛型。
where T : U | 为 T 提供的类型参数必须是为 U 提供的参数或派生自为 U 提供的参数。

&emsp;&emsp;通过约束类型参数，可以增加约束类型及其继承层次结构中的所有类型所支持的允许操作和方法调用的数量。 设计泛型类或方法时，如果要对泛型成员执行除简单赋值之外的任何操作或调用 `System.Object` 不支持的任何方法，则必须对该类型参数应用约束。 例如，基类约束告诉编译器，仅此类型的对象或派生自此类型的对象可用作类型参数。 编译器有了此保证后，就能够允许在泛型类中调用该类型的方法。

可以对同一类型参数应用多个约束，并且约束自身可以是泛型类型，如下所示：

```cs
class EmployeeList<T> where T : Employee, IEmployee, System.IComparable<T>, new()
{
    // ...
}
```

注意：

&emsp;&emsp;在应用 where T : class 约束时，请避免对类型参数使用 == 和 != 运算符，因为这些运算符仅测试引用标识而不测试值相等性。

### 约束多个参数

可以对多个参数应用多个约束，对一个参数应用多个约束，如下例所示：

```cs
class Base { }
class Test<T, U>
    where U : struct
    where T : Base, new()
{ }
```

### 未绑定的类型参数

没有约束的类型参数（如公共类 `SampleClass<T>{}` 中的 T）称为未绑定的类型参数。 未绑定的类型参数具有以下规则：

* 不能使用 != 和 == 运算符，因为无法保证具体的类型参数能支持这些运算符。
* 可以在它们与 System.Object 之间来回转换，或将它们显式转换为任何接口类型。
* 可以将它们与 null 进行比较。 将未绑定的参数与 null 进行比较时，如果类型参数为值类型，则该比较将始终返回 false。

### 类型参数作为约束

&emsp;&emsp;在具有自己类型参数的成员函数必须将该参数约束为包含类型的类型参数时，将泛型类型参数用作约束非常有用，如下例所示：

```cs
public class List<T>
{
    public void Add<U>(List<U> items) where U : T {/*...*/}
}
```

在上述示例中，T 在 Add 方法的上下文中是一个类型约束，而在 List 类的上下文中是一个未绑定的类型参数。

### 非托管约束

从 C# 7.3 开始，可使用 unmanaged 约束来指定类型参数必须为“非托管类型”。

参考：[非托管约束](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#unmanaged-constraint)

### 委托约束

&emsp;&emsp;同样从 C# 7.3 开始，可将 System.Delegate 或 System.MulticastDelegate 用作基类约束。 CLR 始终允许此约束，但 C# 语言不允许。 使用 System.Delegate 约束，用户能够以类型安全的方式编写使用委托的代码。以下代码定义了合并两个同类型委托的扩展方法：

```cs
public static TDelegate TypeSafeCombine<TDelegate>(this TDelegate source, TDelegate target)
    where TDelegate : System.Delegate
    => Delegate.Combine(source, target) as TDelegate;
```

可使用上述方法来合并相同类型的委托：

```cs
Action first = () => Console.WriteLine("this");
Action second = () => Console.WriteLine("that");

var combined = first.TypeSafeCombine(second);
combined();

Func<bool> test = () => true;
// Combine signature ensures combined delegates must
// have the same type.
//var badCombined = first.TypeSafeCombine(test);
```

### 枚举约束

从 C# 7.3 开始，还可指定 System.Enum 类型作为基类约束。 CLR 始终允许此约束，但 C# 语言不允许。 使用 System.Enum 的泛型提供类型安全的编程，缓存使用 System.Enum 中静态方法的结果。

参考：[枚举约束](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters#enum-constraints)

## 泛型类

封闭式构造类型 (`Node<int>`)：客户端代码指定类型参数；
创建开放式构造类型 (`Node<T>`)：不指定类型参数（例如指定泛型基类时）

泛型类可继承自具体的封闭式构造或开放式构造基类：

```cs
class BaseNode { }
class BaseNodeGeneric<T> { }

// concrete type
class NodeConcrete<T> : BaseNode { }

//closed constructed type
class NodeClosed<T> : BaseNodeGeneric<int> { }

//open constructed type 
class NodeOpen<T> : BaseNodeGeneric<T> { }
```

1,非泛型类（即，具体类）可继承自封闭式构造基类，但不可继承自开放式构造类或类型参数，因为运行时客户端代码无法提供实例化基类所需的类型参数。

```cs
//No error
class Node1 : BaseNodeGeneric<int> { }

//Generates an error
//class Node2 : BaseNodeGeneric<T> {}

//Generates an error
//class Node3 : T {}
```

2,继承自开放式构造类型的泛型类必须对非此继承类共享的任何基类类型参数提供类型参数，如下方代码所示：

```cs
class BaseNodeMultiple<T, U> { }

//No error
class Node4<T> : BaseNodeMultiple<T, int> { }

//No error
class Node5<T, U> : BaseNodeMultiple<T, U> { }

//Generates an error
//class Node6<T> : BaseNodeMultiple<T, U> {}
```

3,继承自开放式构造类型的泛型类必须指定作为基类型上约束超集或表示这些约束的约束：

```cs
class NodeItem<T> where T : System.IComparable<T>, new() { }
class SpecialNodeItem<T> : NodeItem<T> where T : System.IComparable<T>, new() { }
```

4,泛型类型可使用多个类型参数和约束，如下所示：

```cs
class SuperKeyType<K, V, U>
    where U : System.IComparable<U>
    where V : new()
{ }
```

如果一个泛型类实现一个接口，则该类的所有实例均可强制转换为该接口。

## 泛型接口

参考：[泛型接口](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/generic-interfaces)

## 泛型方法

参考：[泛型方法](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/generic-methods)

## 泛型和数组

参考：[泛型和数组](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/generics-and-arrays)

## 泛型委托

参考：[泛型委托](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/generic-delegates)

委托可以定义它自己的类型参数。 引用泛型委托的代码可以指定类型参数以创建封闭式构造类型，就像实例化泛型类或调用泛型方法一样，如以下示例中所示：

```cs
public delegate void Del<T>(T item);
public static void Notify(int i) { }

Del<int> m1 = new Del<int>(Notify);

// C# 2.0 版具有一种称为方法组转换的新功能，适用于具体委托类型和泛型委托类型,使你能够使用此简化语法编写上一行：
Del<int> m2 = Notify;
```

在泛型类中定义的委托可以用类方法使用的相同方式来使用泛型类类型参数。

```cs
class Stack<T>
{
    T[] items;
    int index;

    public delegate void StackDelegate(T[] items);
}
```

引用委托的代码必须指定包含类的类型参数，如下所示：

```cs
private static void DoWork(float[] items) { }

public static void TestStack()
{
    Stack<float> s = new Stack<float>();
    Stack<float>.StackDelegate d = DoWork;
}
```

根据典型设计模式定义事件时，泛型委托特别有用，因为发件人参数可以为强类型，无需在它和 Object 之间强制转换。

```cs
delegate void StackEventHandler<T, U>(T sender, U eventArgs);

class Stack<T>
{
    public class StackEventArgs : System.EventArgs { }
    public event StackEventHandler<Stack<T>, StackEventArgs> stackEvent;

    protected virtual void OnStackChanged(StackEventArgs a)
    {
        stackEvent(this, a);
    }
}

class SampleClass
{
    public void HandleStackChange<T>(Stack<T> stack, Stack<T>.StackEventArgs args) { }
}

public static void Test()
{
    Stack<double> s = new Stack<double>();
    SampleClass o = new SampleClass();
    s.stackEvent += o.HandleStackChange;
}
```

参考：

[https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/generics/)