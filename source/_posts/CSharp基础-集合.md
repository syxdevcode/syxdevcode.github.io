---
title: CSharp基础-集合
date: 2018-12-27 08:59:34
tags:
- DotNet
- CSharp
- 集合与泛型
- CSharp基础
categories: 
- 集合与泛型
---
# CSharp基础-集合

集合是.NET FCL(Framework Class Library)中很重要的一部分。.net的集合诸如（`System.Array`类以及 `System.Collections`命名空间）数组、列表、队列、堆栈、哈希表、字典甚至（`System.Data`下)`DataSet`、`DataTable`，还有2.0中加入的集合的泛型版本（`System.Collections.Generic`和 `System.Collections.ObjectModel`）。4.0中引入的有效线程安全操作的集合（`System.Collections.Concurrent`）。

## 集合接口

![28211801-aeaddc8243394d34851a127aad881a29.png](/img/28211801-aeaddc8243394d34851a127aad881a29.png)

### IEnumerable 和 IEnumberator

```cs
public interface IEnumerator
{
    bool MoveNext();
    object Current {  get; }
    void Reset();
}
```

IEnumerator定义了遍历集合的基本方法，以便实现单向向前的访问集合中的每一个元素。而IEnumerable只有一个方法GetEnumerator即得到遍历器。

```cs
public interface IEnumerable
{
    IEnumerator GetEnumerator();
}
```

注意：我们经常用的foreach即是一种语法糖，实际上还是调用Enumerator里面的Current和MoveNext实现的遍历功能。

```cs
List<string> list = new List<string>()
{
    "test1",
    "test2",
    "test3",
    "test4",
    "test5"
};

// Iterate the list by using foreach
foreach (var buddy in list)
{
    Console.WriteLine(buddy);
}

// Iterate the list by using enumerator
List<string>.Enumerator enumerator = list.GetEnumerator();
while (enumerator.MoveNext())
{
    Console.WriteLine(enumerator.Current);
}
```

foreach和enumerator到IL中最后都会被翻译成enumerator的MoveNext和Current。

IEnumerable支持foreach语句。

通过yield来实现返回enumerator:

```cs
public class TestList : IEnumerable
{
    private string[] data = new string[]
    {
        "test1",
        "test2",
        "test3",
        "test4",
        "test5"
    };

    public IEnumerator GetEnumerator()
    {
        foreach (var str in data)
        {
            yield return str;
        }
    }
}

private static void Main(string[] args)
{
    var testLists = new TestList();
    foreach (var str in testLists)
    {
        Console.WriteLine(str);
    }

    Console.Read();
}
```

### `ICollection<T>`和 `ICollection`

`ICollection` 接口是 `System.Collections` 命名空间中类的基接口，`ICollection<T>`是所有泛型版本集合的基接口。所有的的集合类都直接或间接的继承他们。

`ICollection` 又继承 `IEnumerable`，来提供方便的枚举功能。`ICollection` 比 `IEnumerable` 多支持一些功能，不仅仅只提供基本的遍历功能，还包括：

* 统计集合和元素个数
* 获取元素的下标
* 判断是否存在
* 添加元素到未尾
* 移除元素等等。

`ICollection` 与 `ICollection<T>` 略有不同，`ICollection` 不提供编辑集合的功能，即Add和Remove。包括检查元素是否存在Contains也不支持。

### IList<T>和IList

`IList` 则是直接继承自 `ICollection` 和 `IEnumerable`。所以它包括两者的功能。

## 选择集合

![微信截图_20181227105636.png](/img/微信截图_20181227105636.png)

![微信截图_20181227110233.png](/img/微信截图_20181227110233.png)

详细请参考：[https://docs.microsoft.com/zh-cn/dotnet/standard/collections/](https://docs.microsoft.com/zh-cn/dotnet/standard/collections/)

注意非泛型类集合在.NET 2.0时代被泛型类集合代替。

### Dictionary<TKey,TValue>

注解：将项存储为键/值对以通过键进行快速查找

```cs
非泛型集合选项：Hashtable（根据键的哈希代码组织的键/值对的集合。）

线程安全或不可变集合选项：
ConcurrentDictionary<TKey,TValue>
ReadOnlyDictionary<TKey,TValue>
ImmutableDictionary<TKey,TValue>
```

### List<T>

注解：按索引访问项

```cs
非泛型集合选项：
Array
ArrayList

线程安全或不可变集合选项：
ImmutableList<T>
ImmutableArray
```

### Queue<T>

注解：使用项先进先出 (FIFO)

```cs
非泛型集合选项：Queue

线程安全或不可变集合选项：
ConcurrentQueue<T>
ImmutableQueue<T>
```

### Stack<T>

注解：使用数据后进先出 (LIFO)

```cs
非泛型集合选项：Stack

线程安全或不可变集合选项：
ConcurrentStack<T>
ImmutableStack<T>
```

### LinkedList<T>

注解：按顺序访问项

### SortedList<TKey,TValue>

注解：已排序的集合

```cs
非泛型集合选项：SortedList

线程安全或不可变集合选项：
ImmutableSortedDictionary<TKey,TValue>
ImmutableSortedSet<T>
```

### HashSet<T> 和 SortedSet<T>

注解：数学函数的一个集

`HashSet<T> `类提供高性能的设置操作。 集是不包含重复元素的集合，其元素无特定顺序。
`SortedSet<T>` 对象在插入和删除元素时维护排序顺序，而不会影响性能。 不允许重复元素。 不支持更改现有项的排序值，这可能导致意外行为。
```cs
线程安全或不可变集合选项：
ImmutableHashSet<T>
ImmutableSortedSet<T>
```

### ObservableCollection<T>

注解：删除集合中的项或向集合添加项时接收通知。 （实现 INotifyPropertyChanged 和 INotifyCollectionChanged）

参考：

[C#集合类型大盘点](https://www.cnblogs.com/jesse2013/p/CollectionsInCSharp.html)

[https://docs.microsoft.com/zh-cn/dotnet/standard/collections/](https://docs.microsoft.com/zh-cn/dotnet/standard/collections/)

[.NET集合总结](https://www.cnblogs.com/forcertain/archive/2013/04/23/3038214.html)