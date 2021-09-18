---
title: DotNetCore对象池ObjectPool-编程
date: 2021-09-17 16:03:25
tags:
- CSharp
- DotNetCore
categories:
- CSharp
---

转载：

[对象池在 .NET (Core)中的应用[1]: 编程篇](https://www.cnblogs.com/artech/p/object-pool-01.html)

&emsp;&emsp;DotNet 提供有效的自动化垃圾回收机制，.NET让开发人员不在关心对象的生命周期，但实际上很多性能问题都来源于GC。并不说.NET的GC有什么问题，而是对象生命周期的跟踪和管理本身是需要成本的，不论交给应用还是框架来做，都会对性能造成影响。在一些对性能比较敏感的应用中，我们可以通过对象复用的方式避免垃圾对象的产生，进而避免GC因对象回收导致的性能损失。对象池是对象复用的一种常用的方式。.NET提供了一个简单高效的对象池框架，并使用在ASP.NET自身框架中。这个对象池框架由 `Microsoft.Extensions.ObjectPool` NuGet包提供。

## 对象的借与还

&emsp;&emsp;对象池编程方式：需要消费某个对象的时候，我们不会直接创建它，而是选择从对象池中 `借出` 一个对象。一般来说，如果对象池为空，或者现有的对象都正在被使用，它会自动帮助我们完成对象的创建。借出的对象不再使用的时候，我们需要及时将其 `归还` 到对象池中以供后续复用。

在使用.NET的对象池框架时，主要会使用 `ObjectPool<T>`类型，针对池化对象的借与还体现在它的`Get`和`Return`方法中。
<!--more-->
```cs
public abstract class ObjectPool<T> where T: class
{
    public abstract T Get();
    public abstract void Return(T obj);
}
```

**示例：**

添加 `Microsoft.Extensions.ObjectPool` NuGet包。

```cs
// FoobarService具有一个自增整数表示Id属性作为每个实例的唯一标识，
// 静态字段_latestId标识当前分发的最后一个标识
public class FoobarService
{
    internal static int _latestId;
    public int Id { get; }
    public FoobarService() => Id = Interlocked.Increment(ref _latestId);
}
```

```cs
class Program
{
    static async Task Main()
    {
        var objectPool = ObjectPool.Create<FoobarService>();
        while (true)
        {
            Console.Write("Used services: ");
            await Task.WhenAll(Enumerable.Range(1, 3).Select(_ => ExecuteAsync()));
            Console.Write("\n");
        }
        async Task ExecuteAsync()
        {
            var service = objectPool.Get();
            try
            {
                Console.Write($"{service.Id}; ");
                await Task.Delay(1000);
            }
            finally
            {
                objectPool.Return(service);
            }
        }
    }
}
```

实例运行之后会在控制台上输出如下所示的结果，可以看出每轮迭代使用的三个对象都是一样的。每次迭代，它们从对象池中被借出，使用完之后又回到池中供下一次迭代使用。

![19327-20210823194332122-359024594.png](/img/19327-20210823194332122-359024594.png)

## 依赖注入

&emsp;&emsp;采用依赖注入，容器提供的并不是代表对象池的 `ObjectPool<T>` 对象，而是一个 `ObjectPoolProvider`对象。`ObjectPoolProvider` 对象作为对象池的提供者，用来提供针对指定对象类型的 `ObjectPool<T>` 对象。

&emsp;&emsp;.NET提供的大部分框架都提供了针对 `IServiceCollection` 接口的扩展方法来注册相应的服务，但是对象池框架并没有定义这样的扩展方法，所以我们需要采用原始的方式来完成针对 `ObjectPoolProvider` 的注册。如下面的代码片段所示，在创建出 `ServiceCollection` 对象之后，我们通过调用 `AddSingleton` 扩展方法注册了 `ObjectPoolProvider` 的默认实现类型 `DefaultObjectPoolProvider`。

```cs
class Program
{
    static async Task Main()
    {
        var objectPool = new ServiceCollection().AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>()
            .BuildServiceProvider()
            .GetRequiredService<ObjectPoolProvider>()
            .Create<FoobarService>();

        while (true)
        {
            Console.Write("Used services: ");
            await Task.WhenAll(Enumerable.Range(1, 3).Select(_ => ExecuteAsync()));
            Console.Write("\n");
        }
        async Task ExecuteAsync()
        {
            var service = objectPool.Get();
            try
            {
                Console.Write($"{service.Id}; ");
                await Task.Delay(1000);
            }
            finally
            {
                objectPool.Return(service);
            }
        }
    }
}
```

## 池化对象策略

&emsp;&emsp;对象池在默认情况下会帮助我们完成对象的创建工作，它会在对象池无可用对象的时候会调用默认的构造函数来创建提供的对象。如果池化对象类型没有默认的构造函数呢？或者我们希望执行一些初始化操作呢？

&emsp;&emsp;在另一方面，当不在使用的对象被归还到对象池之前，很有可能会执行一些释放性质的操作（比如集合对象在归还之前应该被清空）。还有一种可能是对象有可能不能再次复用（比如它内部维护了一个处于错误状态并无法恢复的网络连接），那么它就不能被释放会对象池。这些需求都可以通过 `IPooledObjectPolicy<T>` 接口表示的池化对象策略来解决。

将上面的 `FoobarService` 更改为以下代码：

```cs
public class FoobarService
{
    internal static int _latestId;
    public int Id { get; }
    private FoobarService() => Id = Interlocked.Increment(ref _latestId);
    public static FoobarService Create() => new FoobarService();
}
```

将构造函数改为私有，并定义了一个静态的工厂方法Create来创建`FoobarService`对象。
。当FoobarService类型失去了默认的无参构造函数之后，演示的程序将无法编译。

![微信截图_20210918133450.png](/img/微信截图_20210918133450.png)

&emsp;&emsp;为了解决这个问题，我们为`FoobarService`类型定义一个代表池化对象策略的FoobarPolicy类型。如代码片段所示，`FoobarPolicy`类型实现了`IPooledObjectPolicy<FoobarService>`接口，实现的Create方法通过调用`FoobarSerivice`类型的静态同名方法完成针对对象的创建。另一个方法Return可以用来执行一些对象归还前的释放操作，它的返回值表示该对象还能否回到池中供后续使用。由于`FoobarService`对象可以被无限次复用，所以实现的Return方法直接返回True。

```cs
public class FoobarPolicy : IPooledObjectPolicy<FoobarService>
{
    public FoobarService Create() => FoobarService.Create();
    public bool Return(FoobarService obj) => true;
}
```

&emsp;&emsp;在调用`ObjectPoolProvider`对象的`Create<T>`方法针对指定的类型创建对应的对象池的时候，我们将一个`IPooledObjectPolicy<T>`对象作为参数，创建的对象池将会根据该对象定义的策略来创建和释放对象。

```cs
class Program
{
    static async Task Main()
    {
        var objectPool = new ServiceCollection().AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>()
            .BuildServiceProvider()
            .GetRequiredService<ObjectPoolProvider>()
            .Create(new FoobarPolicy());
         
         // 省略代码
     }
}
```

## 对象池的大小

对象池容纳对象的数量总归是有限的，默认情况下大小为当前机器处理器数量的2倍。

```cs
class Program
{
    static async Task Main()
    {
        var objectPool = new ServiceCollection().AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>()
            .BuildServiceProvider()
            .GetRequiredService<ObjectPoolProvider>()
            .Create(new FoobarPolicy());
        var poolSize = Environment.ProcessorCount * 2;
        while (true)
        {
            while (true)
            {
                await Task.WhenAll(Enumerable.Range(1, poolSize).Select(_ => ExecuteAsync()));
                Console.WriteLine($"Last service: {FoobarService._latestId}");
            }
        }

        async Task ExecuteAsync()
        {
            var service = objectPool.Get();
            try
            {
                await Task.Delay(1000);
            }
            finally
            {
                objectPool.Return(service);
            }
        }
    }
}
```

运行结果：

![微信截图_20210918134802.png](/img/微信截图_20210918134802.png)

如果将对象的消费率提高，意味着池化的对象将无法满足消费需求，新的对象将持续被创建出来。

```cs
class Program
{
    static async Task Main()
    {
        var objectPool = new ServiceCollection().AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>()
            .BuildServiceProvider()
            .GetRequiredService<ObjectPoolProvider>()
            .Create(new FoobarPolicy());
        var poolSize = Environment.ProcessorCount * 2;
        while (true)
        {
            while (true)
            {
                await Task.WhenAll(Enumerable.Range(1, poolSize + 1)
                    .Select(_ => ExecuteAsync()));
                Console.WriteLine($"Last service: {FoobarService._latestId}");
            }
        }
        …
    }
}
```

由于每次迭代针对对象的需求量是17，但是对象池只能提供16个对象，所以每次迭代都必须额外创建一个新的对象。

结果：

![19327-20210823194333545-1400852638.png](/img/19327-20210823194333545-1400852638.png)

## 对象的释放

&emsp;&emsp;由于对象池容纳的对象数量是有限的，如果现有的所有对象已经被提取出来，它会提供一个新创建的对象。从另一方面讲，我们从对象池得到的对象在不需要的时候总是会还回去，但是对象池可能容不下那么多对象，它只能将其丢弃，被丢弃的对象将最终被GC回收。如果对象类型实现了`IDisposable`接口，在它不能回到对象池的情况下，它的`Dispose`方法应该被立即执行。

```cs
class Program
{
    static async Task Main()
    {
        var objectPool = new ServiceCollection().AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>()
            .BuildServiceProvider()
            .GetRequiredService<ObjectPoolProvider>()
            .Create(new FoobarPolicy());

        while (true)
        {
            Console.Write("Disposed services:");
            await Task.WhenAll(Enumerable.Range(1, Environment.ProcessorCount * 2 + 3).Select(_ => ExecuteAsync()));
            Console.Write("\n");
        }

        async Task ExecuteAsync()
        {
            var service = objectPool.Get();
            try
            {
                await Task.Delay(1000);
            }
            finally
            {
                objectPool.Return(service);
            }
        }
    }
}

public class FoobarService: IDisposable
{
    internal static int _latestId;
    public int Id { get; }
    private FoobarService() => Id = Interlocked.Increment(ref _latestId);
    public static FoobarService Create() => new FoobarService();
    public void Dispose() => Console.Write($"{Id}; ");
}
```

每次迭代消费的19个对象，只有16个能够正常回归对象池，有三个将被丢弃并最终被GC回收。

结果：

![微信截图_20210918135923.png](/img/微信截图_20210918135923.png)
