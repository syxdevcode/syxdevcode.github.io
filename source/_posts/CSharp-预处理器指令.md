---
title: CSharp-预处理器指令
date: 2019-10-14 13:06:11
tags:
- CSharp
- CSharp基础
categories: 
- CSharp基础
---
# CSharp-预处理器指令

## 预处理器指令

预处理器指令（preprocessor directive）：用于在 C# 源代码中嵌入的编译器命令，告诉C#编译器要编译哪些代码，并指出如何处理特定的错误和警告。C#预处理器指令还可以告诉C#编辑器有关代码组织的信息。

选中 项目 ，右键 属性，在 `生成` 标签：

![QQ截图20191014141323.png](/img/QQ截图20191014141323.png)

```cs
#if DEBUG
        Console.WriteLine("DEBUG");
#endif

#if TRACE
        Console.WriteLine("TRACE");
#endif
```

具体预处理器指令参考：

```cs
#if
#else
#elif
#endif
#define
#undef
#warning
#error
#line
#region
#endregion
#pragma
#pragma warning
#pragma checksum
```

## ConditionalAttribute

指示编译器，除非定义了指定的有条件编译符号，否则，应忽略方法调用或属性。

注意：

#if DEBUG：此处的代码在发布时甚至不会到达IL。

[Conditional("DEBUG")]：这个代码将到达IL，但是当设置为DEBUG时，才会被调用。

```cs
[Conditional("DEBUG")]
public void DoSomething() { }

public void Foo()
{
    DoSomething(); //Code compiles and is cleaner, DoSomething always
                   //exists, however this is only called during DEBUG.
}
```

参考：

[C# 预处理器指令](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/preprocessor-directives/)

[如何：使用跟踪和调试执行有条件编译](https://docs.microsoft.com/zh-cn/dotnet/framework/debug-trace-profile/how-to-compile-conditionally-with-trace-and-debug)

[如何：创建、初始化和配置跟踪开关](https://docs.microsoft.com/zh-cn/dotnet/framework/debug-trace-profile/how-to-create-initialize-and-configure-trace-switches)

[-define（C# 编译器选项）](https://docs.microsoft.com/zh-cn/dotnet/csharp/language-reference/compiler-options/define-compiler-option)

[管理项目和解决方案属性](https://docs.microsoft.com/zh-cn/visualstudio/ide/managing-project-and-solution-properties?view=vs-2019)

[#if DEBUG vs. Conditional(“DEBUG”)](https://stackoverflow.com/questions/3788605/if-debug-vs-conditionaldebug)