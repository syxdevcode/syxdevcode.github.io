---
title: CSharp基础-认识反射-下
date: 2019-01-13 10:50:33
tags:
- DotNet
- CSharp
- 反射
- CSharp基础
categories: 
- 反射
---
# CSharp基础-认识反射-下

## 访问自定义特性

参考：[访问自定义特性](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/accessing-custom-attributes)

### 仅反射上下文

加载到仅反射上下文中的代码无法执行。在仅反射上下文中加载和检查自定义特性，使用 `CustomAttributeData` 类。通过使用静态 `CustomAttributeData.GetCustomAttributes` 方法的相应重载来获取此类的实例。

### 执行上下文

用于查询执行上下文中的特性的主要反射方法是 `MemberInfo.GetCustomAttributes` 和 `Attribute.GetCustomAttributes`。

自定义特性的可访问性根据附加该特性的程序集来进行检查。

`Assembly.GetCustomAttributes(Boolean)` 等方法检查类型参数的可见性和可访问性。 只有包含用户定义类型的程序集中的代码才能使用 `GetCustomAttributes` 检索该类型的自定义特性。

自定义特性反射模型可能会在定义类型的程序集外泄漏用户定义类型的实例。 为了防止客户端发现关于用户定义的自定义特性类型的信息，请将该类型的成员定义为非公共成员。

```cs
using System;

public class ExampleAttribute : Attribute
{
    private string stringVal;

    public ExampleAttribute()
    {
        stringVal = "This is the default string.";
    }

    public string StringValue
    {
        get { return stringVal; }
        set { stringVal = value; }
    }
}

[Example(StringValue="This is a string.")]
class Class1
{
    public static void Main()
    {
        System.Reflection.MemberInfo info = typeof(Class1);
        foreach (object attrib in info.GetCustomAttributes(true))
        {
            Console.WriteLine(attrib);
        }
    }
}
```

## 指定完全限定的类型名称

参考：[指定完全限定的类型名称](https://docs.microsoft.com/zh-cn/dotnet/framework/reflection-and-codedom/specifying-fully-qualified-type-names)
