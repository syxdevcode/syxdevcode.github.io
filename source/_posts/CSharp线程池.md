---
title: C#线程池
date: 2020-01-13 14:49:00
tags:
- CSharp
- DotNet
- DotNetCore
- 线程池
categories: 
- 线程池
---
## 线程池

"线程池”就是用来存放“线程”的对象池。

作用：因为创建一个线程的代价较高，因此我们使用线程池设法复用线程。

## CLR线程池

在.NET中，CLR线程和操作系统线程对应，您可以简单地认为.NET中的Thread对象便封装了一个操作系统线程，并附带一些托管环境下所需要的数据（如GC Handle）。而CLR线程池便是存放这些CLR线程的对象池。Thread对象只有当真正Start了之后，CLR才会创建一个操作系统线程与它绑定。

ThreadPool类的两个静态方法：QueueUserWorkItem和UnsafeUserQueueWorkItem向CLR线程池中添加任务（一个WorkCallback委托对象），这两个方法的区别，在于前者会收集调用方的ExecutionContext，也就是保留了的当前线程的执行信息（如认证或语言文化等），使任务最终会在“创建”时刻的环境中执行——后者就不会。因此，如果比较两个方法的绝对性能，Unsafe方法会略胜一筹。但是平时还是建议使用QueueUserWorkItem方法，因为保留执行上下文会避免很多麻烦事情，且这点性能损耗其实算不上什么。

注：

ASP.NET在得到一个请求后，也会将这个请求处理的任务交由CLR线程池去执行——请注意，它们最多只是添加任务而已，并不表示任务会立即执行。所有添加到CLR线程池的任务都会在合适的时候得以执行——可能马上，也可能要稍等片刻，甚至更久。简单的概括说来，便是线程池内有空闲的线程，或线程池所管理的线程数量还没有达到上限的时候。如果有空闲的线程，线程池就会立即让它领取一个任务执行。如果是第二种情况，线程池便会创建新的Thread对象。由于让操作系统管理太多线程反而会造成性能下降，因此CLR线程池会有一个上限。

对于ASP.NET应用程序来说，CLR线程池容量代表了应用程序最多可以同时执行的请求数量。对于托管在IIS上的ASP.NET执行环境来说，这个值由全局配置决定。这个配置在machine.config文件中system.web/processModel节点中，为maxWorkerThreads属性，它决定了为单个处理器分配的线程数。如果这个值为40，且机器上拥有4个处理器（2 * 2CPU），那么这台机器目前的配置表示在同一时刻，ASP.NET可以同时处理160个请求。

既然有最大值，也就相应有了最小值，它代表了CLR线程池“总是会保留”的最少线程数量。由于线程会占用资源，如在默认情况下，每个线程将获得1MB大小的栈空间3。所以如果在系统中保留太多空闲线程对资源也是一种浪费。因此，CLR线程池在使用大量线程处理完大量任务之后，也会逐步地释放线程，直至到达最小值。CLR线程池的最小线程数量确保了在任务数量较少的情况下，新来的任务可以立即执行，从而省去了创建新线程的时间。在普通应用程序中这个值为“处理器数 * 1”，而在ASP.NET应用程序中这个值配置在machine.config文件中system.web/processModel节点的minWorkerThreads属性中4。

对于processModel节点的数据，ASP.NET只会读取machine.config中的全局配置信息，这意味着我们不能使用web.config为不同应用程序配置不同的参数。如果我们要实现应用程序级别的配置，那么必须使用ThreadPool类中提供的API进行设置。

CLR线程池限制了线程的创建速度不超过每秒2个。这样，即使在某个瞬时获得了大量的任务，CLR线程池也可以使用相对较少的线程来完成所有工作。

但是，还有一种情况也值得考虑。例如，对于一个比较繁忙的Web应用程序来说，一打开便会涌入大量的连接。由于线程的创建速度有限，因此可以执行的请求数量也只能慢慢增加。对于这种您预料到会产生大量线程，而且忙碌状况会持续一段时间的情况，限制线程的创建速度反而会带来损伤效率。这时，您就可以手动设置CLR线程池的最小线程数量。如果此时CLR线程池中拥有的线程数量较少，那么系统就会立即创建一定数量的线程来达到这个最小值。设置和获取CLR线程池最小线程数量的接口为：

```cs
public static class ThreadPool
{
    public static void GetMinThreads(out int workerThreads, out int completionPortThreads);
    public static bool SetMinThreads(int workerThreads, int completionPortThreads);
    public static void GetAvailableThreads(out int workerThreads, out int completionPortThreads);
}
```

参考：[ThreadPool 类](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.threadpool?view=netcore-3.1)

注意：

无论是设置还是获取到的这些数值，都与处理器数量没有任何关系了。也就是说，在一台2 * 2CPU的机器上运行一个普通的.NET应用程序时：

* 调用GetMaxThreads方法将获得1000，表示CLR线程池最大容量为1000（250 * 4），而不是250。
* 调用SetMinThreads并传入100，表示CLR线程池所拥有的最小线程数量为100，而不是400（100 * 4）。

## 独立线程池

在一个.NET应用程序中会有一个CLR线程池，可以使用ThreadPool类中的静态方法来使用这个线程池。整个进程内部几乎所有的任务都会依赖这个线程池。由于开发人员对于统一的线程池无法做到精确控制，因此在一些特别的需要就无法满足了。举个最常见例子：控制运算能力。

我们可以简单的认为，在同样的环境下，一个任务使用的线程数量越多，它所获得的运算能力就比另一个线程数量较少的任务要来得多。运算能力自然就涉及到任务执行的快慢。

可以设想一下，有一个生产任务，和一个消费任务，它们使用一个队列做临时存储。在理想情况下，生产和消费的速度应该保持相同，这样可以带来最好的吞吐量。如果生产任务执行较快，则队列中便会产生堆积，反之消费任务就会不断等待，吞吐量也会下降。因此，在实现的时候，我们往往会为生产任务和消费任务分别指派独立的线程池，并且通过增加或减少线程池内线程数量来条件运算能力，使生产和消费的步调达到平衡。

如果需要在同一进程内创建多个线程池，可以借助 [SmartThreadPool](https://github.com/amibar/SmartThreadPool)

## IO线程池

IO线程池便是为异步IO服务的线程池。

BeginGetResponse将发起一个利用IOCP的异步IO操作，并在结束时调用HandleAsyncCallback回调函数。那么，这个回调函数是由哪里的线程执行的呢？没错，就是传说中“IO线程池”的线程。.NET在一个进程中准备了两个线程池，除了上篇文章中所提到的CLR线程池之外，它还为异步IO操作的回调准备了一个IO线程池。IO线程池的特性与CLR线程池类似，也会动态地创建和销毁线程，并且也拥有最大值和最小值。

只可惜，IO线程池也仅仅是那“一整个”线程池，CLR线程池的缺点IO线程池也一应俱全。例如，在使用异步IO方式读取了一段文本之后，下一步操作往往是对其进行分析，这就进入了计算密集型操作了。但对于计算密集型操作来说，如果使用整个IO线程池来执行，我们无法有效的控制某项任务的运算能力。

参考：

[浅谈线程池（上）：线程池的作用及CLR线程池](https://www.cnblogs.com/JeffreyZhao/archive/2009/07/22/thread-pool-1-the-goal-and-the-clr-thread-pool.html)

[浅谈线程池（中）：独立线程池的作用及IO线程池](https://www.cnblogs.com/JeffreyZhao/archive/2009/07/24/thread-pool-2-dedicate-pool-and-io-pool.html)

[浅谈线程池（下）：相关试验及注意事项](https://www.cnblogs.com/JeffreyZhao/archive/2009/10/20/thread-pool-3-lab.html)
