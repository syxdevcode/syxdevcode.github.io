---
title: DotNetCore对象池ObjectPool-扩展
date: 2021-09-18 14:50:12
tags:
- CSharp
- DotNetCore
categories:
- CSharp
---

转载：

[对象池在 .NET (Core)中的应用[3]: 扩展篇](https://www.cnblogs.com/artech/p/object-pool-03.html)

&emsp;&emsp;原则上所有的引用类型对象都可以通过对象池来提供，但是在具体的应用中需要权衡是否值得用。虽然对象池能够通过对象复用的方式避免GC，但是它存储的对象会耗用内存，如果对象复用的频率很小，使用对象池是不值的。如果某个小对象的使用周期很短，能够确保GC在第0代就能将其回收，这样的对象其实也不太适合放在对象池中，因为第0代GC的性能其实是很高的。除此之外，对象释放到对象池之后就有可能被其他线程提取出来，如果释放的时机不对，有可能造成多个线程同时操作同一个对象。总之，我们在使用之前得考虑当前场景是否适用对象池，在使用的时候严格按照 `有借有还`、`不用才还` 的原则。

## 池化集合

&emsp;&emsp;一个 `List<T>` 对象内部会使用一个数组来保存列表元素。数组是定长的，所以`List<T>`有一个最大容量（体现为它的`Capacity`属性）。当列表元素数量超过数组容量时，必须对列表对象进行扩容，即创建一个新的数组并将现有的元素拷贝进去。当前元素越多，需要执行的拷贝操作就越多，对性能的影响自然就越大。

&emsp;&emsp;如果我们创建`List<T>`对象，并在其中不断地添加对象，有可能会导致多次扩容，所以如果能够预知元素数量，我们在创建`List<T>`对象时应该指定一个合适的容量。但是很多情况下，列表元素数量是动态变化的，我们可以利用对象池来解决这个问题。

&emsp;&emsp;接下来我们通过一个简单的实例来演示一下如何采用对象池的方式来提供一个`List<Foobar>`对象，元素类型`Foobar`如下所示。为了能够显式控制列表对象的创建和归还，我们自定义了如下这个表示池化对象策略的`FoobarListPolicy`。通过《设计篇》针对对象池默认实现的介绍，我们知道直接继承`PooledObjectPolicy<T>`类型比实现`IPooledObjectPolicy<T>`接口具有更好的性能优势。

```cs
public class FoobarListPolicy : PooledObjectPolicy<List<Foobar>>
{
    private readonly int _initCapacity;
    private readonly int _maxCapacity;

    public FoobarListPolicy(int initCapacity, int maxCapacity)
    {
        _initCapacity = initCapacity;
        _maxCapacity = maxCapacity;
    }
    public override List<Foobar> Create() => new List<Foobar>(_initCapacity);
    public override bool Return(List<Foobar> obj)
    {
        if(obj.Capacity <= _maxCapacity)
        {
            obj.Clear();
            return true;
        }        
        return false;
    }
}

public class Foobar
{
    public int Foo { get; }
    public int Bar { get; }
    public Foobar(int foo, int bar)
    {
        Foo = foo;
        Bar = bar;
    }
}
```

&emsp;&emsp;如代码片段所示，我们在`FoobarListPolicy`类型中定义了两个字段，`_initCapacity`字段表示列表创建时指定的初始容量，另一个`_maxCapacity`则表示对象池存储列表的最大容量。

&emsp;&emsp;之所以要限制列表的最大容量，是为了避免复用几率很少的大容量列表常驻内存。在实现的`Create`方法中，我们利用初始容量创建出`List<Foobar>`对象。在`Return`方法中，我们先将待回归的列表清空，然后根据其当前容量决定是否要将其释放到对象池。

&emsp;&emsp;下面的程序演示了采用对象池的方式来提供`List<Foobar>`列表。如代码片段所示，我们在调用`ObjectPoolProvider`对象的`Create<T>`创建代表对象池的`ObjectPool<T>`对象时，指定了作为池化对象策略的`FoobarListPolicy`对象。我们将初始和最大容量设置成`1K（1024）`和`1M（1024*1024）`。

&emsp;&emsp;我们利用对象池提供了一个`List<Foobar>`对象，并在其中添加了`10000`个元素。如果这段代码执行的频率很高，对整体的性能是有提升的。

```cs
class Program
{
    static void Main()
    {
        var objectPool = new ServiceCollection()
            .AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>()
            .BuildServiceProvider()
            .GetRequiredService<ObjectPoolProvider>()
            .Create(new FoobarListPolicy(1024, 1024*1024));

        string json;
        var list = objectPool.Get();
        try
        {
            list.AddRange(Enumerable.Range(1, 1000).Select(it => new Foobar(it, it)));
            json = JsonConvert.SerializeObject(list);
        }
        finally
        {
            objectPool.Return(list);
        }
    }
}
```

## 池化StringBuilder

&emsp;&emsp;频繁涉及针对字符串拼接的操作，应该使用`StringBuilder`以获得更好的性能。实际上，`StringBuilder`对象自身也存在类似于列表对象的扩容问题，所以最好的方式就是利用对象池的方式来复用它们。

对象池框架针对`StringBuilder`对象的池化提供的原生支持，我们接下来通过一个简单的示例来演示具体的用法。

```cs
class Program
{
    static void Main()
    {
        var objectPool = new ServiceCollection()
            .AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>()
            .BuildServiceProvider()
            .GetRequiredService<ObjectPoolProvider>()
            .CreateStringBuilderPool(1024, 1024*1024);

        var builder = objectPool.Get();
        try
        {
            for (int index = 0; index < 100; index++)
            {
                builder.Append(index);
            }
            Console.WriteLine(builder);
        }
        finally
        {
            objectPool.Return(builder);
        }
    }
}
```

&emsp;&emsp;如上面的代码片段所示，我们直接可以调用 `ObjectPoolProvider` 的 `CreateStringBuilderPool` 扩展方法就可以得到针对 `StringBuilder` 的对象池对象（类型为 `ObjectPool<StringBuilder>`）。我们上面演示实例一样，我们指定的也是 `StringBuilder` 对象的初始和最大容量。池化 `StringBuilder` 对象的核心体现在对应的策略类型上，即如下这个 `StringBuilderPooledObjectPolicy` 类型。

```cs
public class StringBuilderPooledObjectPolicy : PooledObjectPolicy<StringBuilder>
{
    public int InitialCapacity { get; set; } = 100;
    public int MaximumRetainedCapacity { get; set; } = 4 * 1024;

    public override StringBuilder Create()=> new StringBuilder(InitialCapacity);
    public override bool Return(StringBuilder obj)
    {
        if (obj.Capacity > MaximumRetainedCapacity)
        {
            return false;
        }
        obj.Clear();
        return true;
    }
}
```

&emsp;&emsp;可以看出它的定义和我们前面定义的`FoobarListPolicy`类型如出一辙。在默认情况下，池化`StringBuilder`对象的初始化和最大容量分别为`100`和`5096`。如下所示的是`ObjectPoolProvider`用于创建`ObjectPool<StringBuilder>`对象的两个`CreateStringBuilderPool`扩展方法的定义。

```cs
public static class ObjectPoolProviderExtensions
{
    public static ObjectPool<StringBuilder> CreateStringBuilderPool( this ObjectPoolProvider provider)
        => provider.Create(new StringBuilderPooledObjectPolicy());       

    public static ObjectPool<StringBuilder> CreateStringBuilderPool( this ObjectPoolProvider provider, int initialCapacity, int maximumRetainedCapacity)
    {
        var policy = new StringBuilderPooledObjectPolicy()
        {
            InitialCapacity = initialCapacity,
            MaximumRetainedCapacity = maximumRetainedCapacity,
        };
        return provider.Create(policy);
    }
}
```

## `ArrayPool<T>`

&emsp;&emsp;接下来介绍的和前面的内容没有什么关系，但同属于我们常用对象池使用场景。我们在编程的时候会大量使用到集合，集合类型（像基于链表的集合除外）很多都采用一个数组作为内部存储，所以会有前面所说的扩容问题。如果这个数组很大，还会造成GC的压力。我们在前面已经采用池化集合的方案解决了这个问题，其实这个问题还有另外一种解决方案。

&emsp;&emsp;在很多情况下，当我们需要创建一个对象的时候，实际上需要的一段确定长度的连续对象序列。假设我们将数组对象进行池化，当我们需要一段定长的对象序列的时候，从池中提取一个长度大于所需长度的可用数组，并从中截取可用的连续片段（一般从头开始）就可以了。在使用完之后，我们无需执行任何的释放操作，直接将数组对象归还到对象池中就可以了。这种基于数组的对象池使用方式可以利用`ArrayPool<T>`来实现。

```cs
public abstract class ArrayPool<T>
{    
    public abstract T[] Rent(int minimumLength);
    public abstract void Return(T[] array, bool clearArray);

    public static ArrayPool<T> Create();
    public static ArrayPool<T> Create(int maxArrayLength, int maxArraysPerBucket);

    public static ArrayPool<T> Shared { get; }
}
```

&emsp;&emsp;如上面的代码片段所示，抽象类型 `ArrayPool<T>` 同样提供了完成对象池两个基本操作的方法，其中`Rent` 方法从对象池中 `借出` 一个不小于（不是等于）指定长度的数组，该数组最终通过`Return`方法释放到对象池。`Return`方法的`clearArray`参数表示在归还数组之前是否要将其清空，这取决我们针对数组的使用方式。如果我们每次都需要覆盖原始的内容，就没有必要额外执行这种多余操作。

&emsp;&emsp;我们可以通过静态方法`Create`创建一个`ArrayPool<T>`对象。池化的数组并未直接存储在对象池中，长度接近的多个数组会被封装成一个桶（`Bucket`）中，这样的好处是在执行`Rent`方法的时候可以根据指定的长度快速找到最为匹配的数组（大于并接近指定的长度）。对象池存储的是一组`Bucket`对象，允许的数组长度越大，桶的数量越多。`Create`方法除了可以指定数组允许最大长度，还可以指定每个桶的容量。除了调用静态`Create`方法创建一个独占使用的 `ArrayPool<T>` 对象之外，我们可以使用静态属性`Shared`返回一个应用范围内共享的`ArrayPool<T>`对象。`ArrayPool<T>`的使用非常方便，如下的代码片段演示了一个读取文件的实例。

```cs
class Program
{
    static async Task Main()
    {
        using var fs = new FileStream("test.txt", FileMode.Open);
        var length = (int)fs.Length;
        var bytes = ArrayPool<byte>.Shared.Rent(length);
        try
        {
            await fs.ReadAsync(bytes, 0, length);
            Console.WriteLine(Encoding.Default.GetString(bytes, 0, length));
        }
        finally
        {
            ArrayPool<byte>.Shared.Return(bytes);
        }
    }
}
```

## `MemoryPool<T>`

数组是对托管堆中用于存储同类对象的一段连续内存的表达，而另一个类型 `Memory<T>` 则具有更加广泛的应用，因为它不仅仅可以表示一段连续的托管（Managed）内存，还可以表示一段连续的Native内存，甚至线程堆栈内存。具有如下定义的 `MemoryPool<T>` 表示针对 `Memory<T>` 类型的对象池。

```cs
public abstract class MemoryPool<T> : IDisposable
{    
    public abstract int MaxBufferSize { get; }
    public static MemoryPool<T> Shared { get; }

    public void Dispose();
    protected abstract void Dispose(bool disposing);
    public abstract IMemoryOwner<T> Rent(int minBufferSize = -1);
}

public interface IMemoryOwner<T> : IDisposable
{
    Memory<T> Memory { get; }
}
```

`MemoryPool<T>`和`ArrayPool<T>`具有类似的定义，比如通过静态属性`Shared`获取当前应用全局共享的`MemoryPool<T>`对象，通过`Rent`方法从对象池中借出一个不小于指定大小的`Memory<T>`对象。不同的是，`MemoryPool<T>`的Rent方法并没有直接返回一个`Memory<T>`对象，而是一个封装了该对象的`IMemoryOwner<T>`对象。`MemoryPool<T>`也没有定义一个用来释放`Memory<T>`对象的`Reurn`方法，这个操作是通过`IMemoryOwner<T>`对象的`Dispose`方法完成的。如果采用`MemoryPool<T>`，前面针对`ArrayPool<T>`的演示实例可以改写成如下的形式。

```cs
class Program
{
    static async Task Main()
    {
        using var fs = new FileStream("test.txt", FileMode.Open);
        var length = (int)fs.Length;
        using (var memoryOwner = MemoryPool<byte>.Shared.Rent(length))
        {
            await fs.ReadAsync(memoryOwner.Memory);
            Console.WriteLine(Encoding.Default.GetString( memoryOwner.Memory.Span.Slice(0,length)));
        }                
    }
}
```
