---
title: CSharp-ValueTask解析
date: 2019-11-25 14:28:13
tags:
- DotNet
- CSharp
- Task
- CSharp基础
- .net异步与并行
categories: 
- .net异步与并行
---
## 结构vs类，栈vs堆

&emsp;&emsp;在托管.NET世界中，您应该关注两个内存区域：栈和堆。堆是在其中分配所有对象的托管内存（无论何时调用 `new YourClass`）。重要的方面是它与任何线程都不相关或没有连接。一个线程可以分配一个对象并将其传递给另一线程。只要垃圾收集器没有收集该对象，该对象就会在堆上。

&emsp;&emsp;栈以不同的方式工作。它被分配给特定的线程（每个线程的堆栈大小有限）。栈具有框架，可用于分配一些内存，例如用于在调用过程中创建并分配给变量的结构值，例如，通过调用`Guid.NewGuid()`。

&emsp;&emsp;堆分配的内存可用于在线程之间传输某些数据。由于栈是分配给特定线程的，因此不能用于传输某些数据。另一方面，如果您要分配少量内存，则堆栈将更快一些，因为它是线程本地的，并且不会陷入启动垃圾收集器等缓慢的路径中。

## Task

`Task` 在 .NET Framework 4 中 `System.Threading.Tasks` 命名空间被引入，在 C#5 以及 .NET Framwrok 4.5 中的异步编程模式引入了 `async/await`。

`Task` 的核心就是 `promise`，它表示最终完成的一些操作。当初始化一个操作时，返回一个 `Task`，当操作完成时，`Task` 会完成。

特点：

`Task` 很灵活，并且有很多好处。例如你可以通过多个消费者并行等待多次。你可以存储一个到字典中，以便后面的任意数量的消费者等待，它允许为异步结果像缓存一样使用。如果场景需要，你可以阻塞一个等待完成。并且你可以在这些任务上编写和使用各种操作（就像组合器），例如 WhenAny 操作，它可以异步等待第一个操作完成。

然而，这种灵活性对于大多数情况下是不需要的：仅仅只是调用异步操作并且等待结果：

```cs
TResult result = await SomeOperationAsync();
UseResult(result);
```

副作用:

Task 是一个类。作为一个类，就是说任意操作创建一个 Task 都会分配一个对象，越来越多的对象都会被分配，所以 GC 操作也会越来越频繁，也就会消耗越来越多的资源，本来它应该是去做其他事的。

## ValueTask

.NET CORE 2.0 引入了一个新类型 `ValueTask<TResult>`，在 `System.Threading.Tasks.Extensions` 命名空间中。`ValueTask<TResult>` 作为结构体引入的，它是 `TResult` 或 `Task<TResult>` 包装器。

实例可以等待或 `Task<TResult>` 使用 `AsTask` 转换为。 `ValueTask<TResult>` 实例仅可等待一次, 并且在实例完成之前, 使用者可能不会调用 `GetAwaiter()`。

永远不应对 `ValueTask<TResult>`实例执行以下操作:

```cs
等待实例多次。
多次AsTask调用。
使用/操作未完成/使用多次 .Result.GetAwaiter().GetResult()
使用多种方法来使用此实例。
```

如果执行上述任一操作, 则结果是不确定的。

### 同步完成 (synchronous completion)

它能从异步方法返回并且如果这个方法同步成功完成，不需要分配任何内存：只简单的初始化这个 `ValueTask<TResult>` 结构体，它返回 `TResult`。

只有当该方法异步完成时，`Task<TResult>` 才需要进行分配， 创建包装该 `ValueTask <TResult>` 的实例（以最小化 `ValueTask <TResult>`的大小，并用于优化成功路径，异步方法未处理的异常故障也将分配一个 `Task<TResult>`，所以 `ValueTask<TResult>` 可以简单地包裹该 `Task<TResult>`, 而并不是 随身携带附加字段以存储异常）。

### 异步完成 (asynchronous completion)

在.NET Core 2.1，引入 `IValueTaskSource<TResult>` 支持池化和重用。

.NET Core 2.1为 `ValueTask` 添加了另一个构造函数：

```cs
public readonly struct ValueTask<TResult>
{
    public ValueTask(IValueTaskSource<TResult> source, short token)
}
```

token：令牌参数，该值确保将 `IValueTaskSource` 返回到池后不会使用 `ValueTask`。有效地，此令牌值将传递到 `IValueTaskSource` 实现的每个方法，该方法能够检查调用者是否滥用 `ValueTask`。

```cs
public interface IValueTaskSource<out TResult>
{
    ValueTaskSourceStatus GetStatus(short token);
    void OnCompleted(Action<object> continuation, object state, short token, ValueTaskSourceOnCompletedFlags flags);
    TResult GetResult(short token);
}
```

参与：

[ValueTask<TResult> 结构](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.tasks.valuetask-1?view=netcore-3.0)

[Understanding the Whys, Whats, and Whens of ValueTask](https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/)

[Task, Async Await, ValueTask, IValueTaskSource and how to keep your sanity in modern .NET world](https://blog.scooletz.com/2018/05/14/task-async-await-valuetask-ivaluetasksource-and-how-to-keep-your-sanity-in-modern-net-world/)