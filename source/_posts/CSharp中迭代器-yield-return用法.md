---
title: 'CSharp中迭代器-yield-return用法'
date: 2018-11-14 10:59:41
tags:
- DotNet
- CSharp
- CSharp基础
categories: 
- CSharp基础
---

# CSharp中迭代器yield-return用法

## 迭代器方法

迭代器方法有一个重要限制：在同一方法中不能同时使用 return 语句和 yield return 语句。

此限制通常不是问题。 可以选择在整个方法中使用 yield return，或选择将原始方法分成多个方法，一些使用 return，另一些使用 yield return。

使用 yield return 上下文关键字定义迭代器方法。

yield关键字用于遍历循环中，`yield return`用于返回`IEnumerable<T>`；
`yield break`用于终止循环遍历。

## 简单理解

````csharp
public static bool onOff = false;
public static IEnumerable ForExample()
{
    yield return "1";  // 第一次调用时执行  
    yield return "2";  // 第二次调用时执行  
    if (onOff)         // 第三次调用时执行  
    {
        // 执行 yield break 之后不再执行下面语句  
        yield break;
    }
    // 否则,onOff为 false  
    yield return "3";  // 第四次调用时执行  
    yield return "4";  // 第五次调用时执行  
}
````

## 实例

以下int类型的集合，需要打印出所有值大于2的元素。

```` csharp
static List<int> GetInitialData()
{
  return new List<int>(){1,2,3,4};
}
````

### 不使用yield return的实现

````csharp
static IEnumerable<int> FilterWithoutYield()
{
    List<int> result = new List<int>();
    foreach (int i in GetInitialData())
    {
        if (i > 2)
        {
            result.Add(i);
        }
    }
    return result;
}
````

客户端调用：

````csharp
foreach (var item in FilterWithoutYield())
{
    Console.WriteLine(item);
}
Console.ReadKey();
````

结果：3，4

### 使用yeild return实现

````csharp
static IEnumerable<int> FilterWithYield()
{
    foreach (int i in GetInitialData())
    {
        if (i > 2)
        {
            yield return i;
        }
    }
    yield break; // 执行 yield break 之后不再执行下面语句
    Console.WriteLine("这里的代码不执行");
}
````

````csharp
foreach (var item in FilterWithYield())
{
    Console.WriteLine(item);
}
Console.ReadKey();
````

结果：3，4

### 总结

第一种方法（不使用yield），是把结果集全部加载到内存中再遍历；

第二种方法（使用yield），客户端每调用一次，yield return就返回一个值给客户端，是"按需供给"。

使用yield return为什么能保证每次循环遍历的时候从前一次停止的地方开始执行呢？

--因为，编译器会生成一个状态机来维护迭代器的状态。

简单地说，当希望获取一个`IEnumerable<T>`类型的集合，而不想把数据一次性加载到内存，就可以考虑使用yield return实现"按需供给"。

参考：

[https://docs.microsoft.com/zh-cn/dotnet/csharp/iterators](https://docs.microsoft.com/zh-cn/dotnet/csharp/iterators)

[C#中yield return用法分析](http://phpstudy.php.cn/b.php/97674.html)

[C#中 yield return 与 yield break](https://blog.csdn.net/Golden_Shadow/article/details/7204226)