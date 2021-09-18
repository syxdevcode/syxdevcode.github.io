---
title: DotNetCore对象池ObjectPool-设计
date: 2021-09-18 14:02:59
tags:
- CSharp
- DotNetCore
categories:
- CSharp
---

转载：

[对象池在 .NET (Core)中的应用[2]: 设计篇](https://www.cnblogs.com/artech/p/object-pool-02.html)

对象池模型由三个核心对象构成，它们分别是表示

* 对象池的 `ObjectPool<T>` 对象；
* 对象值提供者的`ObjectPoolProvider`对象；
* 控制池化对象创建与释放行为的 `IPooledObjectPolicy<T>` 对象。

## `IPooledObjectPolicy<T>`

&emsp;&emsp;池化对象策略的 `IPooledObjectPolicy<T>` 对象不仅仅帮助我们创建对象，还可以帮助我们执行一些对象回归对象池之前所需的回收操作，对象最终能否回到对象池中也受它的控制。

<!--more-->
如下面的代码片段所示，`IPooledObjectPolicy<T>` 接口定义了两个方法，`Create` 方法用来创建池化对象，对象回归前需要执行的操作体现在 `Return` 方法上，该方法的返回值决定了指定的对象是否应该回归对象池。抽象类 `PooledObjectPolicy<T>`实现了该接口，我们一般将它作为自定义策略类型的基类。

```cs
public interface IPooledObjectPolicy<T>
{
    T Create();
    bool Return(T obj);
}

public abstract class PooledObjectPolicy<T> : IPooledObjectPolicy<T>
{
    protected PooledObjectPolicy(){}

    public abstract T Create();
    public abstract bool Return(T obj);
}
```

&emsp;&emsp; 我们默认使用的是如下这个 `DefaultPooledObjectPolicy<T>` 类型，由于它直接通过反射来创建池化对象，所以要求泛型参数T必须有一个公共的默认无参构造函数。它的Return方法直接返回True，意味着提供的对象可以被无限制地复用。

```cs
public class DefaultPooledObjectPolicy<T> : PooledObjectPolicy<T> where T: class, new()
{
    public override T Create() => Activator.CreateInstance<T>();
    public override bool Return(T obj) => true;
}
```

## `ObjectPool<T>`

&emsp;&emsp;对象池通过 `ObjectPool<T>` 对象表示。如下面的代码片段所示，`ObjectPool<T>` 是一个抽象类，池化对象通过Get方法提供给我们，我们在使用完之后调用 `Return` 方法将其释放到对象池中以供后续复用。

```cs
public abstract class ObjectPool<T> where T: class
{
    protected ObjectPool(){}

    public abstract T Get();
    public abstract void Return(T obj);
}
```

### `DefaultObjectPool<T>`

&emsp;&emsp;默认使用的对象池体现为一个 `DefaultObjectPool<T>` 对象，由于针对对象池的绝大部分实现就体现这个类型中。对象池具有固定的大小，并且默认的大小为处理器个数的2倍。我们假设对象池的大小为N，那么 `DefaultObjectPool<T>` 对象会如下图所示的方式使用一个单一对象和一个长度为`N-1`的数组来存放由它提供的 `N` 个对象。

![19327-20210825083118230-2060370000.png](/img/19327-20210825083118230-2060370000.png)

&emsp;&emsp;如下面的代码片段所示，`DefaultObjectPool<T>` 使用字段`_firstItem`用来存放第一个池化对象，余下的则存放在`_items`字段表示的数组中。值得注意的是，这个数组的元素类型并非池化对象的类型 `T`，而是一个封装了池化对象的结构体`ObjectWrapper`。如果该数组元素类型改为引用类型T，那么当我们对某个元素进行复制的时候，运行时会进行类型校验（要求指定对象类型派生于T），无形之中带来了一定的性能损失（值类型数组就不需求进行派生类型的校验）。对象池中存在一些性能优化的细节，这就是其中之一。

```cs
public class DefaultObjectPool<T> : ObjectPool<T> where T : class
{
    private protected T _firstItem;
    private protected readonly ObjectWrapper[]  _items;
    …
    private protected struct ObjectWrapper
    {
        public T Element;
    }
}
```

&emsp;&emsp;`DefaultObjectPool<T>`类型定义了如下两个构造函数。在创建一个`DefaultObjectPool<T>`对象的时候会提供一个`IPooledObjectPolicy<T>`对象并指定对象池的大小。对象池的大小默认设置为处理器数量的2倍体现在第一个构造函数重载中。如果指定的是一个`DefaultPooledObjectPolicy<T>`对象，表示默认池化对象策略的`_isDefaultPolicy`字段被设置成True。因为`DefaultPooledObjectPolicy<T>`对象的Return方法总是返回True，并且没有任何具体的操作，所以在将对象释放回对象池的时候就不需要调用 `Return` 方法了，这是第二个性能优化的细节。

```cs
public class DefaultObjectPool<T> : ObjectPool<T> where T : class
{    
    private protected T  _firstItem;
    private protected readonly ObjectWrapper[] _items;
    private protected readonly IPooledObjectPolicy<T> _policy;
    private protected readonly bool  _isDefaultPolicy;
    private protected readonly PooledObjectPolicy<T>  _fastPolicy;
    
    public DefaultObjectPool(IPooledObjectPolicy<T> policy) : this(policy, Environment.ProcessorCount * 2)
    {}

    public DefaultObjectPool(IPooledObjectPolicy<T> policy, int maximumRetained)
    {
        _policy  = policy ;
        _fastPolicy = policy as PooledObjectPolicy<T>;
        _isDefaultPolicy = IsDefaultPolicy();
        _items  = new ObjectWrapper[maximumRetained - 1];

        bool IsDefaultPolicy()
        {
            var type = policy.GetType();
            return type.IsGenericType && type.GetGenericTypeDefinition() == typeof(DefaultPooledObjectPolicy<>);
        }
    }

    [MethodImpl(MethodImplOptions.NoInlining)]
    private T Create() => _fastPolicy?.Create() ?? _policy.Create();
}
```

&emsp;&emsp;从第二个构造函数的定义可以看出，指定的`IPooledObjectPolicy<T>`对象除了会赋值给`_policy`字段之外，如果提供的是一个`PooledObjectPolicy<T>`对象，该对象还会同时赋值给另一个名为`_fastPolicy`的字段。在进行池化对象的提取和释放时，`_fastPolicy`字段表示的池化对象策略会优先选用，这个逻辑体现在`Create`方法上。因为调用类型的方法比调用接口方法具有更好的性能（所以该字段才会命名为`_fastPolicy`），这是第三个性能优化的细节。这个细节还告诉我们在自定义池化对象策略的时候，最好将`PooledObjectPolicy<T>`作为基类，而不是直接实现`IPooledObjectPolicy<T>`接口。

&emsp;&emsp;如下所示的是重写的`Get`和`Return`方法的定义。用于提供池化对象的`Get`方法很简单，它会采用原子操作使用`Null`将`_firstItem`字段表示的对象 `替换` 下来，如果该字段不为 `Null`，那么将其作为返回的对象，反之它会遍历数组的每个 `ObjectWrapper` 对象，并使用 `Null` 将其封装的对象 `替换` 下来，第一个成功替换下来的对象将作为返回值。如果所有 `ObjectWrapper` 对象封装的对象都为Null，意味着所有对象都被 `借出` 或者尚未创建，此时返回创建的新对象了。

```cs
public class DefaultObjectPool<T> : ObjectPool<T> where T : class
{    
    public override T Get()
    {
        var item = _firstItem;
        if (item == null || Interlocked.CompareExchange(ref _firstItem, null, item) != item)
        {
            var items = _items;
            for (var i = 0; i < items.Length; i++)
            {
                item = items[i].Element;
                if (item != null && Interlocked.CompareExchange(ref items[i].Element, null, item) == item)
                {
                    return item;
                }
            }
            item = Create();
        }
        return item;
    }    

    public override void Return(T obj)
    {
        if (_isDefaultPolicy || (_fastPolicy?.Return(obj) ?? _policy.Return(obj)))
        {
            if (_firstItem != null || Interlocked.CompareExchange(ref _firstItem, obj, null) != null)
            {
                var items = _items;
                for (var i = 0; i < items.Length && Interlocked.CompareExchange(ref items[i].Element, obj, null) != null; ++i)
                {}
            }
        }
    }
    …
}
```

&emsp;&emsp;将对象释放会对象池的 `Return` 方法，需要判断指定的对象能否释放会对象池中，如果使用的是默认的池化对象策略，答案是肯定的，否则只能通过调用`IPooledObjectPolicy<T>`对象的`Return`方法来判断。从代码片段可以看出，这里依然会优先选择`_fastPolicy`字段表示的 `PooledObjectPolicy<T>` 对象以获得更好的性能。

&emsp;&emsp;在确定指定的对象可以释放回对象之后，如果`_firstItem`字段为`Null`，`Return`方法会采用原子操作使用指定的对象将其 `替换` 下来。如果该字段不为 `Null` 或者原子替换失败，该方法会便利数组的每个`ObjectWrapper` 对象，并采用原子操作将它们封装的空引用替换成指定的对象。整个方法会在某个原子替换操作成功或者整个便利过程结束之后返回。

&emsp;&emsp;`DefaultObjectPool<T>` 之所有使用一个数组附加一个单一对象来存储池化对象，是因为针对单一字段的读写比针对数组元素的读写具有更好的性能。从上面给出的代码可以看出，不论是Get还是Return方法，优先选择的都是`_firstItem`字段。如果池化对象的使用率不高，基本上使用的都会是该字段存储的对象，那么此时的性能是最高的。

### `DisposableObjectPool<T>`

&emsp;&emsp;当池化对象类型实现了`IDisposable`接口的情况下，如果某个对象在回归对象池的时候，对象池已满，该对象将被丢弃。与此同时，被丢弃对象的`Dispose`方法将立即被调用。但是这种现象并没有在`DefaultObjectPool<T>`类 型的代码中体现出来，这是为什么呢？实际上 `DefaultObjectPool<T>` 还有如下这个名为 `DisposableObjectPool<T>` 的派生类。

&emsp;&emsp;如代码片段可以看出，表示池化对象类型的泛型参数T要求实现`IDisposable`接口。如果池化对象类型实现了`IDisposable`接口，通过默认`ObjectPoolProvider`对象创建的对象池就是一个`DisposableObjectPool<T>` 对象。

```cs
internal sealed class DisposableObjectPool<T> : DefaultObjectPool<T>, IDisposable where T : class
{
    private volatile bool _isDisposed;
    public DisposableObjectPool(IPooledObjectPolicy<T> policy) : base(policy)
    {}

    public DisposableObjectPool(IPooledObjectPolicy<T> policy, int maximumRetained) : base(policy, maximumRetained)
    {}

    public override T Get()
    {
        if (_isDisposed)
        {
            throw new ObjectDisposedException(GetType().Name);
        }
        return base.Get();
    }

    public override void Return(T obj)
    {
        if (_isDisposed || !ReturnCore(obj))
        {
            DisposeItem(obj);
        }
    }

    private bool ReturnCore(T obj)
    {
        bool returnedToPool = false;
        if (_isDefaultPolicy || (_fastPolicy?.Return(obj) ?? _policy.Return(obj)))
        {
            if (_firstItem == null && Interlocked.CompareExchange(ref _firstItem, obj, null) == null)
            {
                returnedToPool = true;
            }
            else
            {
                var items = _items;
                for (var i = 0; i < items.Length && !(returnedTooPool = Interlocked.CompareExchange(ref items[i].Element, obj, null) == null); i++)
                {}
            }
        }
        return returnedTooPool;
    }

    public void Dispose()
    {
        _isDisposed = true;
        DisposeItem(_firstItem);
        _firstItem = null;

        ObjectWrapper[] items = _items;
        for (var i = 0; i < items.Length; i++)
        {
            DisposeItem(items[i].Element);
            items[i].Element = null;
        }
    }

    private void DisposeItem(T item)
    {
        if (item is IDisposable disposable)
        {
            disposable.Dispose();
        }
    }
}
```

&emsp;&emsp;从上面代码片段可以看出，`DisposableObjectPool<T>`自身类型也实现了`IDisposable`接口，它会在`Dispose`方法中调用目前对象池中的每个对象的`Dispose`方法。用于提供池化对象的`Get`方法除了会验证自身的`Disposed`状态之外，并没有特别之处。当对象未能成功回归对象池，通过调用该对象的`Dispose`方法将其释放的操作体现在重写的`Return`方法中。

## ObjectPoolProvider

&emsp;&emsp;表示对象池的 `ObjectPool<T>` 对象是通过 `ObjectPoolProvider` 提供的。

&emsp;&emsp;如下面的代码片段所示，抽象类 `ObjectPoolProvider` 定义了两个重载的`Create<T>`方法，抽象方法需要指定具体的池化对象策略。另一个重载由于采用默认的池化对象策略，所以要求对象类型具有一个默认无参构造函数。

```cs
public abstract class ObjectPoolProvider
{
    public ObjectPool<T> Create<T>() where T : class, new() => Create<T>(new DefaultPooledObjectPolicy<T>());
    public abstract ObjectPool<T> Create<T>(IPooledObjectPolicy<T> policy) where T : class;
}
```

&emsp;&emsp;在前面的示例演示中，我们使用的是如下这个`DefaultObjectPoolProvider`类型。如代码片段所示，`DefaultObjectPoolProvider`派生于抽象类`ObjectPoolProvider`，在重写的`Create<T>`方法中，它会根据泛型参数T是否实现`IDisposable`接口分别创建`DisposableObjectPool<T>`和`DefaultObjectPool<T>`对象。

```cs
public class DefaultObjectPoolProvider : ObjectPoolProvider
{
    public int MaximumRetained { get; set; } = Environment.ProcessorCount * 2;
    public override ObjectPool<T> Create<T>(IPooledObjectPolicy<T> policy) => typeof(IDisposable).IsAssignableFrom(typeof(T))
        ? new DisposableObjectPool<T>(policy, MaximumRetained) : new DefaultObjectPool<T>(policy, MaximumRetained);
}
```

&emsp;&emsp;`DefaultObjectPoolProvider`类型定义了一个标识对象池大小的`MaximumRetained`属性，采用处理器数量的两倍作为默认容量也体现在这里。这个属性并非只读，所以我们可以利用它根据具体需求调整提供对象池的大小。在ASP.NET应用中，我们基本上都会采用依赖注入的方式利用注入的`ObjectPoolProvider`对象来创建针对具体类型的对象池。

我们在《编程篇》还演示了另一种创建对象池的方式，那就是直接调用 `ObjectPool` 类型的静态`Create<T>`方法，该方法的实现体现在如下所示的代码片段中。

```cs
public static class ObjectPool
{
    public static ObjectPool<T> Create<T>(IPooledObjectPolicy<T> policy) where T: class, new()
        => new DefaultObjectPoolProvider().Create<T>(policy ?? new DefaultPooledObjectPolicy<T>());
}
```

总得来说，整个对象池的设计模型是一个简单、高效并且具有可扩展性的对象池框架，该模型涉及的几个核心接口和类型体现在如下图所示的UML中。

![19327-20210825083119059-1150432674.png](/img/19327-20210825083119059-1150432674.png)
