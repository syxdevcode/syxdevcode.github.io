---
title: DotNet基础-并行编程之数据并行
date: 2018-12-25 08:23:01
tags:
- DotNet
- CSharp
- Task
- .net异步与并行
categories: 
- .net异步与并行
---
任务并行主要处理任务，而数据并行是要从直观上移除任务，用一种更高级的抽象--并行循环，来替代任务。也就是说，并行的源不是算法的代码，而是算法所操作的数据。

## Parallel

### Parallel.For

```cs
for (int j = 1; j < 4; j++)
{
    Console.WriteLine("\n第{0}次比较", j);

    ConcurrentBag<int> bag = new ConcurrentBag<int>();

    var watch = Stopwatch.StartNew();

    watch.Start();

    for (int i = 0; i < 20000000; i++)
    {
        bag.Add(i);
    }

    Console.WriteLine("串行计算：集合有:{0},总共耗时：{1}", bag.Count, watch.ElapsedMilliseconds);

    GC.Collect();

    bag = new ConcurrentBag<int>();

    watch = Stopwatch.StartNew();

    watch.Start();

    Parallel.For(0, 20000000, i =>
    {
        bag.Add(i);
    });

    Console.WriteLine("并行计算：集合有:{0},总共耗时：{1}", bag.Count, watch.ElapsedMilliseconds);

    GC.Collect();
}
```

比较结果：
![微信截图_20181225092626.png](/img/微信截图_20181225092626.png)

注意：

* Parallel.For不支持浮点数的步进，使用的是Int32或Int64，每一次迭代的时候加1
* 使用Parallel.For所迭代的顺序是无法保证的

### Parallel.ForEach

forEach的独到之处就是可以将数据进行分区，每一个小区内实现串行计算，分区采用Partitioner.Create实现。

```cs
for (int j = 1; j < 4; j++)
{
    Console.WriteLine("\n第{0}次比较", j);

    ConcurrentBag<int> bag = new ConcurrentBag<int>();

    var watch = Stopwatch.StartNew();

    watch.Start();

    for (int i = 0; i < 30000000; i++)
    {
        bag.Add(i);
    }

    Console.WriteLine("串行计算：集合有:{0},总共耗时：{1}", bag.Count, watch.ElapsedMilliseconds);

    GC.Collect();

    bag = new ConcurrentBag<int>();

    watch = Stopwatch.StartNew();

    watch.Start();

    // 创建分区的范围是0-3000000
    // Partitioner.Create(0, 3000000, Environment.ProcessorCount)
    //  Environment.ProcessorCount能够获取到当前的硬件线程数
    Parallel.ForEach(Partitioner.Create(0, 30000000), i =>
    {
        for (int m = i.Item1; m < i.Item2; m++)
        {
            bag.Add(m);
        }
    });

    Console.WriteLine("并行计算：集合有:{0},总共耗时：{1}", bag.Count, watch.ElapsedMilliseconds);

    GC.Collect();
}
```

结果：

![微信截图_20181225093224.png](/img/微信截图_20181225093224.png)

### Parallel.Invoke

试图将很多方法并行运行，如果传入的是4个方法，则至少需要4个逻辑内核才能足以让这4个方法并发运行。

注意：

1.即使拥有4个逻辑内核，也不一定能够保证所需要运行的4个方法能够同时启动运行，如果其中的一个内核处于繁忙状态，那么底层的调度逻辑可能会延迟某些方法的初始化执行。

2.通过Parallel.Invoke编写的并发执行代码一定不能依赖与特定的执行顺序，因为它的并发执行顺序也是不定的。

3.使用Parallel.Invoke方法一定要测量运行结果、实现加速比以及逻辑内核的使用率，这点很重要。

4.使用Parallel.Invoke，在运行并行方法前都会产生一些额外的开销，如分配硬件线程等。

```cs
private static void Main(string[] args)
{
    var watch = Stopwatch.StartNew();

    watch.Start();

    Run1();

    Run2();

    Console.WriteLine("我是串行开发，总共耗时:{0}\n", watch.ElapsedMilliseconds);

    watch.Restart();

    Parallel.Invoke(Run1, Run2);

    watch.Stop();

    Console.WriteLine("我是并行开发，总共耗时:{0}", watch.ElapsedMilliseconds);

    Console.ReadLine();
}

private static void Run1()
{
    Console.WriteLine("我是任务一,我跑了3s");
    Thread.Sleep(3000);
}

private static void Run2()
{
    Console.WriteLine("我是任务二，我跑了5s");
    Thread.Sleep(5000);
}
```

![微信截图_20181225094750.png](/img/微信截图_20181225094750.png)

### 中途退出并行循环

ParallelLoopState，该实例提供了Break和Stop方法来帮我们实现。

Break: 通知并行计算尽快的退出循环，比如并行计算正在迭代100，那么break后程序还会迭代所有小于100的。

Stop：并行循环应该尽快停止执行，如果调用Stop时迭代100正在被处理，那么循环无法保证处理完所有小于100的迭代。

```cs
var watch = Stopwatch.StartNew();

watch.Start();

ConcurrentBag<int> bag = new ConcurrentBag<int>();

Parallel.For(0, 20000000, (i, state) =>
{
    if (bag.Count == 1000)
    {
        state.Break();
        return;
    }
    bag.Add(i);
});

Console.WriteLine("当前集合有{0}个元素。", bag.Count);
```

![微信截图_20181225100015.png](/img/微信截图_20181225100015.png)

### 异常处理

```cs
private static void Main(string[] args)
{
    try
    {
        Parallel.Invoke(Run1, Run2);
    }
    catch (AggregateException ex)
    {
        foreach (var single in ex.InnerExceptions)
        {
            Console.WriteLine(single.Message);
        }
    }

    Console.Read();
}

private static void Run1()
{
    Thread.Sleep(3000);
    throw new Exception("我是任务1抛出的异常");
}

private static void Run2()
{
    Thread.Sleep(5000);

    throw new Exception("我是任务2抛出的异常");
}
```

![微信截图_20181225100207.png](/img/微信截图_20181225100207.png)

### 指定硬件线程数量

```cs
var bag = new ConcurrentBag<int>();

ParallelOptions options = new ParallelOptions();

//指定使用的硬件线程数为1
options.MaxDegreeOfParallelism = 1;

Parallel.For(0, 300000, options, i =>
{
    bag.Add(i);
});

Console.WriteLine("并行计算：集合有:{0}", bag.Count);

Console.Read();
```

## 并行Linq（PLINQ）

通过 AsParallel() 扩展方法，将集合从普通的 IEnumerable<T> 改为ParallelQuery<T>。

PLINQ优于Parallel.ForEach的原因在于，PLINQ可以自动将执行查询的线程内部的临时处理结果聚合起来。

PLINQ使用3级处理管道来执行并行查询：

* 1，首先，PLINQ决定需要多少线程来执行并行查询；
* 2，其次，工作站线程从源集合中获取工作块，确保在有锁的情况下访问该工作块。每个线程独立的执行其工作项，并将结果压入本地队列。
* 3，最终，所有本地结果会缓存到单个结果集合中。

```cs
var source = Enumerable.Range(1, 10000);

// Opt in to PLINQ with AsParallel.
var evenNums = from num in source.AsParallel()
               where num % 2 == 0
               select num;
Console.WriteLine("{0} even numbers out of {1} total",
                  evenNums.Count(), source.Count());
// The example displays the following output:
//       5000 even numbers out of 10000 total
```

参考：

[8天玩转并行开发——第一天 Parallel的使用](http://www.cnblogs.com/huangxincheng/archive/2012/04/02/2429543.html)

[C#并行编程-Parallel](https://www.cnblogs.com/woxpp/p/3925094.html)

[C#并行编程-相关概念](http://www.cnblogs.com/woxpp/p/3924476.html)

[https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.tasks.parallel?view=netframework-4.7.2](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.tasks.parallel?view=netframework-4.7.2)

[https://docs.microsoft.com/zh-cn/dotnet/standard/parallel-programming/how-to-specify-the-execution-mode-in-plinq](https://docs.microsoft.com/zh-cn/dotnet/standard/parallel-programming/how-to-specify-the-execution-mode-in-plinq)