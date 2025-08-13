---
title: CSharp中DebuggerStepThrough特性节省Debug时间
date: 2019-04-28 15:03:41
tags:
- CSharp基础
- DotNet
categories: 
- CSharp基础
---
# CSharp中DebuggerStepThrough特性节省Debug时间

`DebuggerStepThroughAttribute`: 指示调试器逐句通过代码，而不是单步执行代码。

只支持类，结构，构造方法，方法。


此属性避免必须进入编译器提供的代码，只进入开发人员提供的代码。例如，如果使用F11（Step Into）键逐步执行代码，则该属性将使步骤的行为类似于编译器提供的代码的F10（Step Over）键。该方法不会被引入，但它将被执行。

例如：

```cs
[DebuggerStepThrough]
public static class Check
{
    [ContractAnnotation("value:null => halt")]
    public static T NotNull<T>(T value, [InvokerParameterName] [NotNull] string parameterName)
    {
        if (value == null)
        {
            throw new ArgumentNullException(parameterName);
        }

        return value;
    } 
}

// 执行代码
partdto part = new partdto();
Check.NotNull(part.PartDto, "partDTO"); // 按F11执行单步调试，则编译器会执行类似F10的行为
```

参考：

[DebuggerStepThroughAttribute Class](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.debuggerstepthroughattribute?view=netframework-4.8)

[仅我的代码](https://docs.microsoft.com/zh-cn/visualstudio/debugger/just-my-code?view=vs-2015)