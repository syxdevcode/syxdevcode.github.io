---
title: DotNet泛型中的协变和逆变
date: 2018-12-27 11:31:55
tags:
- DotNet
- CSharp
- 集合与泛型
- CSharp基础
categories: 
- 集合与泛型
---
协变和逆变都是术语，协变指能够使用比原始指定的派生类型的派生程度更大（更具体的）的类型，逆变指能够使用比原始指定的派生类型的派生程度更小（不太具体的）的类型。

规律总结：

<font color=#ff0000 size=4 face="黑体">“子类”向“父类”转换，即协变；（可以使用的范围变大）
“父类”向“子类”转换，即逆变；（可以使用的范围变小）</font>

## 变体

<font color=#0099ff size=4 face="黑体">协变和逆变统称为“变体”（Variant）。 未标记为协变或逆变的泛型类型参数称为“固定参数” 。</font>

规则：

<font color=#ff0000 size=4 face="黑体">通常，协变类型参数可用作委托的返回类型，而逆变类型参数可用作参数类型。
对于接口，协变类型参数可用作接口的方法的返回类型，而逆变类型参数可用作接口的方法的参数类型。</font>

有关公共语言运行时中变体的事项的简短摘要：

* 在 .NET Framework 4中，`Variant` 类型参数仅限于泛型接口和泛型委托类型。
* 泛型接口或泛型委托类型可以同时具有协变和逆变类型参数。
* 变体仅适用于引用类型；如果为 `Variant` 类型参数指定值类型，则该类型参数对于生成的构造类型是不变的。
* 变体不适用于委托组合。 也就是说，在给定类型 `Action<Derived>` 和 `Action<Base>` 的两个委托的情况下，无法将第二个委托与第一个委托结合起来，尽管结果将是类型安全的。 变体允许将第二个委托分配给类型 Action<Derived>的变量，但只能在这两个委托的类型完全匹配的情况下对它们进行组合。

**Covariance:协变**
使你能够使用比原始指定的类型派生程度更大的类型。

**Contravariance：逆变**
使你能够使用比原始指定的类型更泛型（派生程度更小）的类型。

**Invariance：不变**
这意味着，你只能使用原始指定的类型；固定泛型类型参数既不是协变类型，也不是逆变类型。

## 关键字

**协变类型参数用`out`关键字**

可以将协变类型参数用作属于接口的方法的返回值，或用作委托的返回类型。 但不能将协变类型参数用作接口方法的泛型类型约束。如果接口的方法具有泛型委托类型的参数，则接口类型的协变类型参数可用于指定委托类型的逆变类型参数。

**逆变类型参数用 `in` 关键字**

可以将逆变类型参数用作属于接口的方法的参数类型，或用作委托的参数类型。 也可以将逆变类型参数用作接口方法的泛型类型约束。

<font color=#ff0000 size=4 face="黑体">注意，只有接口类型和委托类型才能具有 变体（Variant） 类型参数。 接口或委托类型可以同时具有协变和逆变类型参数。</font>

C# 不允许违反协变和逆变类型参数的使用规则，也不允许将协变和逆变批注添加到接口和委托类型之外的类型参数中。 MSIL 汇编程序 不执行此类检查，但如果你尝试加载违反规则的类型，则会引发 TypeLoadException 。

## 协变out（泛型修饰符）

利用协变类型参数，你可以执行非常类似于普通的多态性的分配

```cs
IEnumerable<Derived> d = new List<Derived>();
IEnumerable<Base> b = d;
```

**1,协变泛型接口:**

```c#
// Covariant interface.
interface ICovariant<out R> { }

// Extending covariant interface.
interface IExtCovariant<out R> : ICovariant<R> { }

// Implementing covariant interface.
class Sample<R> : ICovariant<R> { }

class Program
{
    static void Test()
    {
        ICovariant<Object> iobj = new Sample<Object>();
        ICovariant<String> istr = new Sample<String>();

        // You can assign istr to iobj because
        // the ICovariant interface is covariant.
        iobj = istr;
    }
}
```

**2，协变泛型委托:**

```C#
// Covariant delegate.
public delegate R DCovariant<out R>();

// Methods that match the delegate signature.
public static Control SampleControl()
{ return new Control(); }

public static Button SampleButton()
{ return new Button(); }

public void Test()
{
    // Instantiate the delegates with the methods.
    DCovariant<Control> dControl = SampleControl;
    DCovariant<Button> dButton = SampleButton;

    // You can assign dButton to dControl
    // because the DCovariant delegate is covariant.
    dControl = dButton;

    // Invoke the delegate.
    dControl();
}
```

## 逆变in（泛型修饰符）

由于 lambda 表达式与其自身所分配到的委托相匹配，因此它会定义一个方法，此方法采用一个类型 Base 的参数且没有返回值。 可以将结果委托分配给类型类型 `Action<Derived>` 的变量，因为 T 委托的类型参数 `Action<T>` 是逆变类型参数。 由于 T 指定了一个参数类型，因此该代码是类型安全代码。

```cs
Action<Base> b = (target) => { Console.WriteLine(target.GetType().Name); };
Action<Derived> d = b;
d(new Derived());
```

**1,逆变泛型接口:**

```c#
// Contravariant interface.
interface IContravariant<in A> { }

// Extending contravariant interface.
interface IExtContravariant<in A> : IContravariant<A> { }

// Implementing contravariant interface.
class Sample<A> : IContravariant<A> { }

class Program
{
    static void Test()
    {
        IContravariant<Object> iobj = new Sample<Object>();
        IContravariant<String> istr = new Sample<String>();

        // You can assign iobj to istr because
        // the IContravariant interface is contravariant.
        istr = iobj;
    }
}
```

**2,逆变泛型委托:**

```c#
// Contravariant delegate.
public delegate void DContravariant<in A>(A argument);

// Methods that match the delegate signature.
public static void SampleControl(Control control)
{ }
public static void SampleButton(Button button)
{ }

public void Test()
{
    // Instantiating the delegates with the methods.
    DContravariant<Control> dControl = SampleControl;
    DContravariant<Button> dButton = SampleButton;

    // You can assign dControl to dButton
    // because the DContravariant delegate is contravariant.
    dButton = dControl;

    // Invoke the delegate.
    dButton(new Button());
}
```

## 泛型接口中的变体

### 具有协变类型参数的泛型接口

从 .NET Framework 4开始，某些泛型接口具有协变类型参数；例如： `IEnumerable<T>`、 `IEnumerator<T>`、 `IQueryable<T>` 和 `IGrouping<TKey,TElement>`。 由于这些接口的所有类型参数都是协变类型参数，因此这些类型参数只用于成员的返回类型。

```cs
using System;
using System.Collections.Generic;

class Base
{
    public static void PrintBases(IEnumerable<Base> bases)
    {
        foreach(Base b in bases)
        {
            Console.WriteLine(b);
        }
    }
}

class Derived : Base
{
    public static void Main()
    {
        List<Derived> dlist = new List<Derived>();

        Derived.PrintBases(dlist);
        IEnumerable<Base> bIEnum = dlist;
    }
}
```

### 具有逆变泛型类型参数的泛型接口

从 .NET Framework 4开始，某些泛型接口具有逆变类型参数；例如： `IComparer<T>`、 `IComparable<T>`和 `IEqualityComparer<T>`。 由于这些接口只具有逆变类型参数，因此这些类型参数只用作接口成员中的参数类型。

`IComparer<T>.Compare` 方法的实现基于 Area 属性的值，所以 `ShapeAreaComparer` 可用于按区域对 Shape 对象排序。
该示例创建 `SortedSet<T>` 对象的 `Circle` ，使用采用 `IComparer<Circle>` 的构造函数。 但是，该对象不传递 `IComparer<Circle>`，而是传递一个用于实现 `ShapeAreaComparer` 的 `IComparer<Shape>` 对象。 当代码需要派生程度较大的类型的比较器 `(Shape)` 时，该示例可以传递派生程度较小的类型的比较器 `(Circle)`，因为 `IComparer<T>` 泛型接口的类型参数是逆变参数。

向 `Circle` 中添加新 `SortedSet<Circle>`对象时，每次将新元素与现有元素进行比较时，都会调用 `IComparer<Shape>.Compare` 对象的`IComparer(Of Shape).Compare` 方法。 方法 (Shape) 的参数类型比被传递的类型 (Circle) 的派生程度小，所以调用是类型安全的。 逆变使 `ShapeAreaComparer` 可以对派生自 Shape的任意单个类型的集合以及混合类型的集合排序。

```cs
using System;
using System.Collections.Generic;

abstract class Shape
{
    public virtual double Area { get { return 0; }}
}

class Circle : Shape
{
    private double r;
    public Circle(double radius) { r = radius; }
    public double Radius { get { return r; }}
    public override double Area { get { return Math.PI * r * r; }}
}

class ShapeAreaComparer : System.Collections.Generic.IComparer<Shape>
{
    int IComparer<Shape>.Compare(Shape a, Shape b)
    {
        if (a == null) return b == null ? 0 : -1;
        return b == null ? 1 : a.Area.CompareTo(b.Area);
    }
}

class Program
{
    static void Main()
    {
        // You can pass ShapeAreaComparer, which implements IComparer<Shape>,
        // even though the constructor for SortedSet<Circle> expects 
        // IComparer<Circle>, because type parameter T of IComparer<T> is
        // contravariant.
        SortedSet<Circle> circlesByArea = 
            new SortedSet<Circle>(new ShapeAreaComparer()) 
                { new Circle(7.2), new Circle(100), null, new Circle(.01) };

        foreach (Circle c in circlesByArea)
        {
            Console.WriteLine(c == null ? "null" : "Circle with area " + c.Area);
        }
    }
}

/* This code example produces the following output:

null
Circle with area 0.000314159265358979
Circle with area 162.860163162095
Circle with area 31415.9265358979
 */
 ```

## 泛型委托中的变体

在 .NET Framework 4中， `Func` 泛型委托（如 `Func<T,TResult>`）具有协变返回类型和逆变参数类型。 `Action` 泛型委托（如 `Action<T1,T2>`）具有逆变参数类型。 这意味着，可以将委托指派给具有派生程度较高的参数类型和（对于 `Func` 泛型委托）派生程度较低的返回类型的变量。

备注：

`Func` 泛型委托的最后一个泛型类型参数指定委托签名中返回值的类型。 该参数是协变的（`out` 关键字），而其他泛型类型参数是逆变的（`in` 关键字）。

## Variant 泛型接口和委托类型的列表

参考：[Variant 泛型接口和委托类型的列表](https://docs.microsoft.com/zh-cn/dotnet/standard/generics/covariance-and-contravariance#list-of-variant-generic-interface-and-delegate-types)

参考：

[泛型中的协变和逆变](https://docs.microsoft.com/zh-cn/dotnet/standard/generics/covariance-and-contravariance)

[out（泛型修饰符）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/out-generic-modifier)

[in（泛型修饰符）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/in-generic-modifier)