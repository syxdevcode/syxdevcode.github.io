---
title: TPL DataFlow笔记（2）
date: 2020-01-10 16:50:01
tags:
- DotNet
- CSharp
- Task
- .net异步与并行
- TPL DataFlow
categories: 
- TPL DataFlow
---
转载：[TPL DataFlow初探（二）](https://www.cnblogs.com/haoxinyue/archive/2013/03/01/2938953.html)

[TPL DataFlow初探（二）](https://syxdevcode.github.io/2020/01/10/TPL%20DataFlow%E7%AC%94%E8%AE%B0%EF%BC%881%EF%BC%89/)简单的介绍了TDF提供的一些Block，通过对这些Block配置和组合，可以满足很多的数据处理的场景。这一篇将继续介绍与这些Block配置的相关类，和挖掘一些高级功能。

在一些Block的构造函数中，我们常常可以看见需要你输入 `DataflowBlockOptions` 类型或者它的两个派生类型`ExecutionDataflowBlockOptions` 和 `GroupingDataflowBlockOptions`。

## DataflowBlockOptions

`DataflowBlockOptions` 有五个属性：`BoundedCapacity`，`CancellationToken`，`MaxMessagesPerTask`，`NameFormat` 和 `TaskScheduler`。

### 用BoundedCapacity来限定容量

这个属性用来限制一个Block中最多可以缓存数据项的数量，大多数Block都支持这个属性，这个值默认是`DataflowBlockOptions.Unbounded = -1`，也就是说没有限制。开发人员可以制定这个属性设置数量的上限。那后面的新数据将会延迟。比如说用一个 `BufferBlock` 连接一个 `ActionBlock`，如果在 `ActionBlock` 上面设置了上限，`ActionBlock` 处理的操作速度比较慢，留在 `ActionBlock` 中的数据到达了上限，那么余下的数据将留在`BufferBlock`中，直到 `ActionBlock` 中的数据量低于上限。这种情况常常会发生在生产者生产的速度大于消费者速度的时候，导致的问题是内存越来越大，数据操作越来越延迟。我们可以通过一个 `BufferBlock` 连接多个`ActionBlock` 来解决这样的问题，也就是负载均衡。一个 `ActionBlock` 满了，就会放到另外一个 `ActionBlock` 中去了。

### 用CancellationToken来取消操作

TPL中常用的类型。在Block的构造函数中放入 `CancellationToken`，Block将在它的整个生命周期中全程监控这个对象，只要在这个Block结束运行（调用Complete方法）前，用 `CancellationToken` 发送取消请求，该Block将会停止运行，如果Block中还有没有处理的数据，那么将不会再被处理。

### 用MaxMessagesPerTask控制公平性

每一个Block内部都是异步处理，都是使用TPL的 `Task`。TDF的设计是在保证性能的情况下，尽量使用最少的任务对象来完成数据的操作，这样效率会高一些，一个任务执行完成一个数据以后，任务对象并不会销毁，而是会保留着去处理下一个数据，直到没有数据处理的时候，Block才会回收掉这个任务对象。但是如果数据来自于多个Source，公平性就很难保证。从其他Source来的数据必须要等到早前的那些Source的数据都处理完了才能被处理。这时我们就可以通过`MaxMessagesPerTask` 来控制。这个属性的默认值还是 `DataflowBlockOptions.Unbounded=-1`，表示没有上限。假如这个数值被设置为1的话，那么单个任务只会处理一个数据。这样就会带来极致的公平性，但是将带来更多的任务对象消耗。

### 用NameFormat来定义Block名称

MSDN上说属性 `NameFormat` 用来获取或设置查询块的名称时要使用的格式字符串。

Block的名字 `Name=string.format(NameFormat, block.GetType ().Name, block.Completion.Id)`。所以当我们输入”{0}”的时候，名字就是 `block.GetType ().Name`，如果我们数据的是”{1}”，那么名字就是`block.Completion.Id`。如果是“{2}”，那么就会抛出异常。

### 用TaskScheduler来调度Block行为

`TaskScheduler` 是非常重要的属性。同样这个类型来源于TPL。每个Block里面都使用 `TaskScheduler` 来调度行为，无论是源Block和目标Block之间的数据传递，还是用户自定义的执行数据方法委托，都是使用的 `TaskScheduler`。如果没有特别设置的话，将使用 `TaskScheduler.Default（System.Threading.Tasks.ThreadPoolTaskScheduler）`来调度。我们可以使用其他的一些继承于 `TaskScheduler` 的类型来设置这个调度器，一旦设置了以后，Block中的所有行为都会使用这个调度器来执行。`.Net Framework 4` 中内建了两个 `Scheduler` ，一个是默认的 `ThreadPoolTaskScheduler` ，另一个是用于UI线程切换的 `SynchronizationContextTaskScheduler` 。如果你使用的Block设计到UI的话，那可以使用后者，这样在UI线程切换上面将更加方便。

.Net Framework 4.5 中，还有一个类型被加入到 `System.Threading.Tasks` 名称空间下：`ConcurrentExclusiveSchedulerPair` 。这个类是两个 `TaskScheduler` 的组合。它提供两个 `TaskScheduler`： `ConcurrentScheduler` 和 `ExclusiveScheduler`；我们可以把这两个 `TaskScheduler` 构造进要使用的Block中。他们保证了在没有排他任务的时候（使用 `ExclusiveScheduler` 的任务），其他任务（使用`ConcurrentScheduler`）可以同步进行，当有排他任务在运行的时候，其他任务都不能运行。其实它里面就是一个读写锁。这在多个Block操作共享资源的问题上是一个很方便的解决方案。

```cs
public ActionBlock<int> readerAB1;

public ActionBlock<int> readerAB2;

public ActionBlock<int> readerAB3;

public ActionBlock<int> writerAB1;

public BroadcastBlock<int> bb = new BroadcastBlock<int>((i) => { return i; });

public void Test()
{
    ConcurrentExclusiveSchedulerPair pair = new ConcurrentExclusiveSchedulerPair();

    readerAB1 = new ActionBlock<int>((i) =>
    {
        Console.WriteLine("ReaderAB1 begin handling." + " Execute Time:" + DateTime.Now);
        Thread.Sleep(500);
    }
    , new ExecutionDataflowBlockOptions() { TaskScheduler = pair.ConcurrentScheduler });

    readerAB2 = new ActionBlock<int>((i) =>
    {
        Console.WriteLine("ReaderAB2 begin handling." + " Execute Time:" + DateTime.Now);
        Thread.Sleep(500);
    }
    , new ExecutionDataflowBlockOptions() { TaskScheduler = pair.ConcurrentScheduler });

    readerAB3 = new ActionBlock<int>((i) =>
    {
        Console.WriteLine("ReaderAB3 begin handling." + " Execute Time:" + DateTime.Now);
        Thread.Sleep(500);
    }
    , new ExecutionDataflowBlockOptions() { TaskScheduler = pair.ConcurrentScheduler });

    writerAB1 = new ActionBlock<int>((i) =>
    {
        Console.ForegroundColor = ConsoleColor.Red;
        Console.WriteLine("WriterAB1 begin handling." + " Execute Time:" + DateTime.Now);
        Console.ResetColor();
        Thread.Sleep(3000);
    }
    , new ExecutionDataflowBlockOptions() { TaskScheduler = pair.ExclusiveScheduler });

    bb.LinkTo(readerAB1);
    bb.LinkTo(readerAB2);
    bb.LinkTo(readerAB3);


    Task.Run(() =>
    {
        while (true)
        {
            bb.Post(1);
            Thread.Sleep(1000);
        }
    });

    Task.Run(() =>
    {
        while (true)
        {
            Thread.Sleep(6000);
            writerAB1.Post(1);
        }
    });

}
```

![01161456-7b8227892cb14d20b5347fa310224619.jpg](/img/01161456-7b8227892cb14d20b5347fa310224619.jpg)

### 用MaxDegreeOfParallelism来并行处理

通常，Block中处理数据都是单线程的，一次只能处理一个数据，比如说 `ActionBlock` 中自定义的代理。使用 `MaxDegreeOfParallelism` 可以让你并行处理这些数据。属性的定义是最大的并行处理个数。如果定义成-1的话，那就是没有限制。用户需要在实际情况中选择这个值的大小，并不是越大越好。如果是平行处理的话，还应该考虑是否有共享资源。

## TDF中的负载均衡

我们可以使用Block很方便的构成一个生产者消费者的模式来处理数据。当生产者产生数据的速度快于消费者的时候，消费者Block的Buffer中的数据会越来越多，消耗大量的内存，数据处理也会延时。这时，我们可以用一个生产者Block连接多个消费者Block来解决这个问题。由于多个消费者Block一定是并行处理，所以对共享资源的处理一定要做同步处理。

### 使用BoundedCapacity属性来实现

当连接多个 `ActionBlock` 的时候，可以通过设置 `ActionBlock` 的 `BoundedCapacity` 属性。当第一个满了，就会放到第二个，第二个满了就会放到第三个。

```cs
public BufferBlock<int> bb = new BufferBlock<int>();

public ActionBlock<int> ab1 = new ActionBlock<int>((i) =>
    {
        Thread.Sleep(1000);
        Console.WriteLine("ab1 handle data" + i + " Execute Time:" + DateTime.Now);
    }
    , new ExecutionDataflowBlockOptions() { BoundedCapacity = 2 });

public ActionBlock<int> ab2 = new ActionBlock<int>((i) =>
    {
        Thread.Sleep(1000);
        Console.WriteLine("ab2 handle data" + i + " Execute Time:" + DateTime.Now);
    }
    , new ExecutionDataflowBlockOptions() { BoundedCapacity = 2 });

public ActionBlock<int> ab3 = new ActionBlock<int>((i) =>
    {
        Thread.Sleep(1000);
        Console.WriteLine("ab3 handle data:" + i + " Execute Time:" + DateTime.Now);
    }
    , new ExecutionDataflowBlockOptions() { BoundedCapacity = 2 });

public void Test()
{
    bb.LinkTo(ab1);
    bb.LinkTo(ab2);
    bb.LinkTo(ab3);

    for (int i = 0; i < 9; i++)
    {
        bb.Post(i);
    }
}
```

![01161504-e9c265982cb54d0eb844ee1a1be1e92d.jpg](/img/01161504-e9c265982cb54d0eb844ee1a1be1e92d.jpg)
