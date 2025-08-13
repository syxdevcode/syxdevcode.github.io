---
title: DotNet 单例模式
date: 2019-03-21 08:33:10
tags:
  - CSharp基础
  - 单例模式
  - DotNet面试题解析
categories:
  - 设计模式
---

单例模式涉及到的基础知识，包括静态构造函数，私有构造函数，锁，延时创建对象, readonly/const 等等。

单例的优点： 1.保证了所有的对象访问的都是同一个实例 2.由于类是由自己类控制实例化的，所以有相应的伸缩性

单例的缺点： 1.额外的系统开销，因为每次使用类的实例的时候，都要检查实例是否存在，可以通过静态实例该解决。 2.无法销毁对象，单例模式的特性决定了只有他自己才能销毁对象实例，但是一般情况下我们都没做这个事情。

<!--more-->

## 版本 1

主要问题，就是线程安全的问题，当 2 个请求同时方式这个类的实例的时候，可以会在同一时间点上都创建一个实例。

```cs
using System;

public sealed class Singleton
{
   private static Singleton instance;

   // 私有构造函数，确保用户在外部不能实例化新的实例
   private Singleton() {}

   public static Singleton Instance
   {
      get
      {
         if (instance == null)
         {
            instance = new Singleton();
         }
         return instance;
      }
   }
}
```

## 版本 2

标记类为 sealed 是好的，可以防止被集成;
在首次访问静态成员之前以及在调用构造函数（如果有）之前，会初始化静态成员。

静态私有字段上通过 new 的形式，来保证在该类第一次被调用的时候创建实例,但是，C#其实并不保证实例创建的时机，因为 C#规范只是在 IL 里标记该静态字段是`BeforeFieldInit`，也就是说静态字段可能在第一次被使用的时候创建，也可能你没使用了，它也帮你创建了。

```cs
public sealed class Singleton
{
   // 在静态私有字段上声明单例
   private static readonly Singleton instance = new Singleton();

   // 私有构造函数，确保用户在外部不能实例化新的实例
   private Singleton(){}

   // 只读属性返回静态字段
   public static Singleton Instance
   {
      get
      {
         return instance;
      }
   }
}
```

## 版本 3

volatile 关键字表示字段可能被多个并发执行线程修改。声明为 `volatile` 的字段不受编译器优化（假定由单个线程访问）的限制。这样可以确保该字段在任何时间呈现的都是最新的值。

<font color=#ff0000 size=4 face="黑体">常见的实现方法，推荐使用</font>

每个线程都会对线程辅助对象 locker 加锁之后再判断实例是否存在，对于这个操作完全没有必要的，因为当第一个线程创建了该类的实例之后，后面的线程此时只需要直接判断（instance==null）为假，此时完全没必要对线程辅助对象加锁之后再去判断，所以上面的实现方式增加了额外的开销，损失了性能，为了改进上面实现方式的缺陷，我们只需要在 lock 语句前面加一句（instance==null）的判断就可以避免锁所增加的额外开销，这种实现方式我们就叫它 “双重锁定”。

```cs
public sealed class Singleton
{
    private static volatile Singleton instance = null;

    // Lock对象，线程安全所用
    private static object syncRoot = new Object();

    private Singleton() { }

    public static Singleton Instance
    {
        get
        {
            if (instance == null)
            {
                lock (syncRoot)
                {
                    if (instance == null)
                        instance = new Singleton();
                }
            }

            return instance;
        }
    }
}
```

## 版本 4

通过加静态构造函数，确保是个延迟初始化的单例。

静态构造函数：用于初始化任何静态数据，或执行仅需执行一次的特定操作。
<font color=#ff0000 size=4 face="黑体">在创建第一个实例或引用任何静态成员之前自动调用静态构造函数。</font>

```cs
public class Singleton
{
    // 因为下面声明了静态构造函数，所以在第一次访问该类之前，new Singleton()语句不会执行
    private static readonly Singleton _instance = new Singleton();

    public static Singleton Instance
    {
        get { return _instance; }
    }

    private Singleton()
    {
    }

    // 声明静态构造函数就是为了删除IL里的BeforeFieldInit标记
    // 以掉静态字段在使用之前被初始化
    static Singleton()
    {
    }
}
```

## 版本 5

版本 4 的变体

```cs
public sealed class Singleton
{
    private Singleton()
    {
    }

    public static Singleton Instance { get { return Nested._instance; } }

    private class Nested
    {
        static Nested()
        {
        }

        internal static readonly Singleton _instance = new Singleton();
    }
}
```

## 版本 6

`Lazy<T>`默认的设置就是线程安全。

```cs
public class Singleton
{
    // 因为构造函数是私有的，所以需要使用lambda
    private static readonly Lazy<Singleton> _instance =
        new Lazy<Singleton>(() => new Singleton());
    // new Lazy<Singleton>(() => new Singleton(), LazyThreadSafetyMode.ExecutionAndPublication);

    private Singleton()
    {
    }

    public static Singleton Instance
    {
        get
        {
            return _instance.Value;
        }
    }
}
```

## 泛型单例

```cs
public abstract class Singleton<T> where T : class
{
    private static readonly Lazy<T> _instance
        = new Lazy<T>(() =>
        {
            var ctors = typeof(T).GetConstructors(
                BindingFlags.Instance
                | BindingFlags.NonPublic
                | BindingFlags.Public);
            if (ctors.Count() != 1)
                throw new InvalidOperationException($"Type {typeof(T)} must have exactly one constructor.");
            var ctor = ctors.SingleOrDefault(c => !c.GetParameters().Any() && c.IsPrivate);
            if (ctor == null)
                throw new InvalidOperationException(
                    $"The constructor for {typeof(T)} must be private and take no parameters.");
            return (T)ctor.Invoke(null);
        });

    public static T Instance => _instance.Value;
}
```

泛型单例类的实现：

```cs
class MySingleton : Singleton<MySingleton>
{
    int _counter;

    public int Counter
    {
        get { return _counter; }
    }

    private MySingleton()
    {
        _counter = 0;
    }

    public void IncrementCounter()
    {
        ++_counter;
    }
    // 线程安全
    public void IncrementCounterLock()
    {
        Interlocked.Increment(ref _counter);
    }
}
```

泛型版本的代码使用的`Lazy<T>`能确保我们在创建单例实例的时候是线程安全的，但是不意味着单例本身是线程安全的，例如：

```cs
static void Main(string[] args)
{
    Parallel.For(0, 100, i =>
    {
        for (int j = 0; j < 1000; ++j)
            MySingleton.Instance.IncrementCounter();
    });

    // Counter的结果是小于100000的
    Console.WriteLine("Counter={0}.", MySingleton.Instance.Counter);
    Console.ReadLine();
}
```

参考：

[大叔手记（10）：别再让面试官问你单例（暨 6 种实现方式让你堵住面试官的嘴）](http://www.cnblogs.com/TomXu/archive/2011/12/19/2291448.html)

[.NET 线程同步之 Volatile 构造](https://blog.csdn.net/daguanjia11/article/details/74612674)
