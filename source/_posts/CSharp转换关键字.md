---
title: CSharp转换关键字
date: 2018-12-14 10:39:56
tags:
- CSharp
- DotNet
- CSharp基础
categories: 
- CSharp基础
---
# CSharp转换关键字

## explicit(显示转换)

`explicit`:必须通过转换来调用的用户定义的类型转换运算符。

&emsp;&emsp;此转换运算符从源类型转换为目标类型。 源类型提供转换运算符。 不同于隐式转换，显式转换运算符必须通过转换的方式来调用。 如果转换操作会导致异常或丢失信息，则应将其标记为 explicit。 这可阻止编译器静默调用可能产生意外后果的转换操作。
省略转换将导致编译时错误 CS0266。

```csharp
struct Digit
{
    byte value;
    public Digit(byte value)
    {
        if (value > 9)
        {
            throw new ArgumentException();
        }
        this.value = value;
    }

    // Define explicit byte-to-Digit conversion operator:
    public static explicit operator Digit(byte b)
    {
        Digit d = new Digit(b);
        Console.WriteLine("conversion occurred");
        return d;
    }
}

class ExplicitTest
{
    static void Main()
    {
        try
        {
            byte b = 3;
            Digit d = (Digit)b; // explicit conversion
        }
        catch (Exception e)
        {
            Console.WriteLine("{0} Exception caught.", e);
        }
    }
}
/*
Output:
conversion occurred
*/
```

## implicit(隐式转换)

`implicit` 关键字用于声明隐式的用户定义类型转换运算符。 如果可以确保转换过程不会造成数据丢失，则可使用该关键字在用户定义类型和其他类型之间进行隐式转换。

```csharp
class Digit
{
    public Digit(double d) { val = d; }
    public double val;
    // ...other members

    // User-defined conversion from Digit to double
    public static implicit operator double(Digit d)
    {
        return d.val;
    }
    //  User-defined conversion from double to Digit
    public static implicit operator Digit(double d)
    {
        return new Digit(d);
    }
}

class Program
{
    static void Main(string[] args)
    {
        Digit dig = new Digit(7);
        //This call invokes the implicit "double" operator
        double num = dig;
        //This call invokes the implicit "Digit" operator
        Digit dig2 = 12;
        Console.WriteLine("num = {0} dig2 = {1}", num, dig2.val);
        Console.ReadLine();
    }
}
/*
Output:
num = 7 dig2 = 12
*/
```

## operator(运算符)

operator作用：

* 1, 重载内置运算符
* 2, 在类或结构声明中提供用户定义的转换

若要在自定义类或结构上重载运算符，可以在相应的类型中创建运算符声明。 重载内置 C# 运算符的运算符声明必须满足以下规则：

* 同时包含 public 和 static 修饰符。
* 包含 operator X，其中 X 是被重载运算符的名称或符号。
* 一元运算符具有一个参数，二元运算符具有两个参数。 在每种情况下，都必须至少有一个参数与声明运算符的类或结构的类型相同。

注：<font color=#0099ff size=4 face="黑体">
1,一元运算符：++，--，!;
2,二元运算符：+，-，* /;
3,三元运算符：a=3>4?3:4
</font>

```csharp
class Fraction
{
    int num, den;
    public Fraction(int num, int den)
    {
        this.num = num;
        this.den = den;
    }

    // overload operator +
    public static Fraction operator +(Fraction a, Fraction b)
    {
        return new Fraction(a.num * b.den + b.num * a.den,
           a.den * b.den);
    }

    // overload operator *
    public static Fraction operator *(Fraction a, Fraction b)
    {
        return new Fraction(a.num * b.num, a.den * b.den);
    }

    // user-defined conversion from Fraction to double
    public static implicit operator double(Fraction f)
    {
        return (double)f.num / f.den;
    }
}

class Test
{
    static void Main()
    {
        Fraction a = new Fraction(1, 2);
        Fraction b = new Fraction(3, 7);
        Fraction c = new Fraction(2, 3);
        Console.WriteLine((double)(a * b + c));
    }
}
/*
Output
0.880952380952381
*/
```

参考：

[https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/explicit](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/keywords/explicit)