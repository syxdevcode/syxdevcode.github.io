---
title: CSharp基础-LINQ标准查询运算符
date: 2018-12-29 10:37:02
tags:
- DotNet
- CSharp
- LINQ
- CSharp基础
categories: 
- LINQ
---
# CSharp基础-LINQ标准查询运算符

&emsp;&emsp;标准查询运算符是组成 LINQ 模式的方法。 这些方法中的大多数都作用于序列；其中序列指其类型实现 `IEnumerable<T>` 接口或 `IQueryable<T>` 接口的对象。 标准查询运算符提供包括筛选、投影、聚合、排序等在内的查询功能。

## 查询表达式语法

参考：

[标准查询运算符的查询表达式语法](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/query-expression-syntax-for-standard-query-operators)

## 按执行方式的分类

参考：

[标准查询运算符按执行方式的分类](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/classification-of-standard-query-operators-by-manner-of-execution)

## 对数据排序

方法名|描述|C# 查询表达式语法
------|----|------
OrderBy|按升序对值排序。| `orderby`
OrderByDescending|按降序对值排序。| `orderby … descending`
ThenBy|按升序执行次要排序。|`orderby …, …`
ThenByDescending| 按降序执行次要排序。| `orderby …, … descending`
Reverse | 反转集合中元素的顺序。| 不适用。

参考：

[对数据排序](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/sorting-data)

## 集运算

参考：

[集运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/set-operations)

## 筛选数据

方法名|描述| C# 查询表达式语法
-----|----|-----
OfType|根据其转换为特定类型的能力选择值。|不适用。
Where|选择基于谓词函数的值。|`where`

参考:

[筛选数据](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/filtering-data)

## 限定符运算

参考：

[限定符运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/quantifier-operations)

## 投影运算

方法名|描述| C# 查询表达式语法
-----|----|-----
选择|投影基于转换函数的值。|`select`
SelectMany|投影基于转换函数的值序列，然后将它们展平为一个序列。|使用多个 `from` 子句

参考：

[投影运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/projection-operations)

## 数据分区

参考：

[数据分区](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/partitioning-data)

## 联接运算

方法名|描述| C# 查询表达式语法
-----|----|-----
联接|根据键选择器函数联接两个序列并提取值对。|`join … in … on … equals …`
GroupJoin|根据键选择器函数联接两个序列，并对每个元素的结果匹配项进行分组。|`join … in … on … equals … into …`

参考：

[联接运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/join-operations)

## 对数据分组

方法名|描述| C# 查询表达式语法
-----|----|-----
GroupBy|对共享通用属性的元素进行分组。 </br>每组由一个 IGrouping<TKey,TElement> 对象表示。|group … by 或 group … by … into …
ToLookup|将元素插入基于键选择器函数的</br>Lookup<TKey,TElement>（一种一对多字典）。|不适用。

参考：

[对数据分组](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/grouping-data)

## 生成运算

参考：

[生成运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/generation-operations)

## 相等运算

参考：

[相等运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/generation-operations)

## 元素运算

参考：

[元素运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/element-operations)

## 转换数据类型

方法名|描述| C# 查询表达式语法
-----|----|-----
AsEnumerable|返回类型化为 IEnumerable<T> 的输入。|不适用。
AsQueryable	|将（泛型）IEnumerable 转换为（泛型）IQueryable。|不适用。
Cast|将集合中的元素转换为指定类型。|使用显式类型化的范围变量。 </br>例如:from string str in words
OfType| 根据其转换为指定类型的能力筛选值。|不适用。
ToArray	|将集合转换为数组。 此方法强制执行查询。|不适用。
ToDictionary|根据键选择器函数将元素放入 Dictionary<TKey,TValue>。</br> 此方法强制执行查询。|不适用。
ToList|将集合转换为 List<T>。 此方法强制执行查询。|不适用。
ToLookup|根据键选择器函数将元素放入 Lookup<TKey,TElement>（一对多字典）。 此方法强制执行查询。|不适用。

参考：

[转换数据类型](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/converting-data-types)

## 串联运算

参考：

[串联运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/concatenation-operations)

## 聚合运算

方法名|描述| C# 查询表达式语法|详细信息
-----|----|-----|----
聚合|对集合的值执行自定义聚合运算。|不适用。|Enumerable.Aggregate</br>Queryable.Aggregate
平均值|计算值集合的平均值。|不适用。|Enumerable.Average</br>Queryable.Average
计数|对集合中元素计数，可选择仅对满足</br>谓词函数的元素计数。|不适用。|Enumerable.Count</br>Queryable.Count
LongCount|对大型集合中元素计数，可选择仅对</br>满足谓词函数的元素计数。|不适用。|Enumerable.LongCount</br>Queryable.LongCount
最大值|确定集合中的最大值。|不适用。|Enumerable.Max</br>Queryable.Max
最小值|确定集合中的最小值。|不适用。|Enumerable.Min</br>Queryable.Min
Sum|对集合中的值求和。|不适用。|Enumerable.Sum</br>Queryable.Sum

参考：

[聚合运算](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/aggregation-operations)

参考：

[标准查询运算符概述 (C#)](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/standard-query-operators-overview)

[https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.enumerable.oftype?view=netframework-4.7.2](https://docs.microsoft.com/zh-cn/dotnet/api/system.linq.enumerable.oftype?view=netframework-4.7.2)