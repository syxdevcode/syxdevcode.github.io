---
title: DotNet基础-异步编程
date: 2018-12-23 11:00:37
tags:
- DotNet
- CSharp
- CSharp基础
- .net异步与并行
categories: 
- .net异步与并行
---
.NET Framework 1.0 引进了 IAsyncResult 模式，也称为 Asynchronous Programming Model (APM)或 Begin/End 模式。 .NET Framework 2.0 增加了 Event-based Asynchronous Pattern (EAP)。 从.NET Framework 4 开始， Task-based Asynchronous Pattern (TAP) 取代了 APM 和 EAP，但能够轻松构建从早期模式中迁移的例程。

<font color=#0099ff size=4 face="黑体">APM异步编程模式下，callback是执行在另一个线程中，不能随易的去更新UI。EAP这种异步编程模式下，事件绑定的方法也是在调用的那个线程中执行的。也就是说解决了异步编程的时候UI交互的问题，而且是在同一个线程中执行。</font>

APM,EAP模式请参考：[https://docs.microsoft.com/zh-cn/dotnet/standard/asynchronous-programming-patterns/asynchronous-programming-model-apm](https://docs.microsoft.com/zh-cn/dotnet/standard/asynchronous-programming-patterns/asynchronous-programming-model-apm)

## 异步编程准则

异步编程的准则是确定所需执行的操作是`I/O-Bound` 还是 `CPU-Bound`。因为这会极大影响代码性能，并可能导致某些构造的误用。

* 如果代码会 `等待` 某些内容，例如数据库中的数据或web资源等,则你的工作是 `I/O-Bound`。
* 如果代码要执行开销巨大的计算，则你的工作是 `CPU-Bound`。

<font color=#0099ff size=4 face="黑体">如果你的工作为 `I/O-Bound`，请使用 `async` 和 `await`（而不使用 Task.Run）。 不应使用任务并行库。
如果你的工作为 `CPU-Bound`，并且你重视响应能力，请使用 async 和 await，并在另一个线程上使用 Task.Run 生成工作。 如果该工作同时适用于并发和并行，则应考虑使用任务并行库。</font>

## 基于任务的异步模式 (TAP)

基于任务的异步模式 (TAP) 以 `System.Threading.Tasks.Task` 命名空间中的 `System.Threading.Tasks.Task<TResult>` 和 `System.Threading.Tasks` 类型为基础，这些类型用于表示任意异步操作。 对于新的开发项目，建议采用 TAP 作为异步设计模式。

C# 5.0引入了2个新关键词：`async`和`await`。然而它大大简化了异步方法的编程。`async`和`await`关键字只是编译器功能。编译器会用Task类创建代码。

### 认识async和await

使用`async`和`await`关键词编写异步代码，具有与同步代码相当的结构和简单性，并且摒弃了异步编程的复杂结构。

`await` 不会开启新的线程，当前线程会一直往下走直到遇到真正的Async方法（比如说`HttpClient.GetStringAsync`），这个方法的内部会用`Task.Run`或者`Task.Factory.StartNew` 去开启线程。如果方法不是.NET为我们提供的`Async`方法，我们需要自己创建Task，才会真正的去创建线程。

如果另一个线程已经执行完毕（即`name.IsCompleted=true`），主线程仍然不用挂起，直接可以拿结果。
如果另一个线程还没有执行完毕（即`name.IsCompleted=false`），那么主线程会挂起等待，直到返回结果为止。

### 解析async和await

#### 异步（async）

使用`async`修饰符标记的方法称为异步方法，异步方法只可以具有以下返回类型：

* 1.Task
* 2.`Task<TResult>`
* 3.void
* 4.从C# 7.0开始，任何具有可访问的`GetAwaiter`方法的类型。`System.Threading.Tasks.ValueTask<TResult>` 类型属于此类实现（需向项目添加`System.Threading.Tasks.Extensions` NuGet 包）。

&emsp;&emsp;异步方法通常包含 `await` 运算符的一个或多个实例，但缺少 `await` 表达式也不会导致生成编译器错误。 如果异步方法未使用 `await` 运算符标记暂停点，那么异步方法会作为同步方法执行，即使有 async 修饰符也不例外，编译器将为此类方法发布一个警告。

#### 等待（await）

`await` 表达式只能在由 `async` 修饰符标记的封闭方法体、`lambda` 表达式或异步方法中出现。在其他位置，它会解释为标识符。

使用`await`运算符的任务只可用于返回 `Task`、`Task<TResult>` 和 `System.Threading.Tasks.ValueType<TResult>` 对象的方法。

<font color=#0099ff size=4 face="黑体">异步方法同步运行，直至到达其第一个 `await` 表达式，此时 `await` 在方法的执行中插入挂起点，会将方法挂起直到所等待的任务完成，然后继续执行`await`后面的代码区域。</font>
<font color=#ff0000 size=4 face="黑体">await 表达式并不阻止正在执行它的线程。 而是使编译器将剩下的异步方法注册为等待任务的延续任务。 控制权随后会返回给异步方法的调用方。 任务完成时，它会调用其延续任务，异步方法的执行会在暂停的位置处恢复。</font>

注意：

* 1.无法等待具有 `void` 返回类型的异步方法，并且无效返回方法的调用方捕获不到异步方法抛出的任何异常。

* 2.异步方法无法声明 in、ref 或 out 参数，但可以调用包含此类参数的方法。 同样，异步方法无法通过引用返回值，但可以调用包含 ref 返回值的方法。

<font color=#ff0000 size=4 face="黑体">await并不是针对于async的方法，而是针对async方法所返回给我们的Task。</font>

##### 不用await关键字，确认Task执行完毕

```cs
static void Main(){
    var task = Task.Run(() =>{
        return GetName();
    });

    task.GetAwaiter().OnCompleted(() =>{
        // 2 秒之后才会执行这里
        var name = task.Result;
        Console.WriteLine("My name is: " + name);
    });

    Console.WriteLine("主线程执行完毕");
    Console.ReadLine();
}

static string GetName(){
    Console.WriteLine("另外一个线程在获取名称");
    Thread.Sleep(2000);
    return "Hello World";
}
```

##### Task.GetAwaiter()和await Task 的区别

* 加上`await`关键字之后，后面的代码会被挂起等待，直到`task`执行完毕有返回值的时候才会继续向下执行，这一段时间主线程会处于挂起状态。
* `GetAwaiter`方法会返回一个`awaitable`的对象（继承了`INotifyCompletion.OnCompleted`方法）我们只是传递了一个委托进去，等`task`完成了就会执行这个委托，但是并不会影响主线程，下面的代码会立即执行。这也是为什么我们结果里面第一句话会是 `主线程执行完毕`！

##### Task如何让主线程挂起等待

```cs
static void Main(){
    var task = Task.Run(() =>{
        return GetName();
    });

    var name = task.GetAwaiter().GetResult();
    Console.WriteLine("My name is:{0}",name);

    Console.WriteLine("主线程执行完毕");
    Console.ReadLine();
}

static string GetName(){
    Console.WriteLine("另外一个线程在获取名称");
    Thread.Sleep(2000);
    return "Hello World";
}
```

`Task.GetAwait()`方法会给我们返回一个`awaitable`的对象，通过调用这个对象的`GetResult`方法就会挂起主线程，当然也不是所有的情况都会挂起。在一开始的时候就启动了另一个线程去执行这个Task，当我们调用它的结果的时候,如果这个Task已经执行完毕，主线程是不用等待可以直接拿其结果的，如果没有执行完毕那主线程就得挂起等待了。

##### await的实质

`await`的实质是在调用`awaitable`对象的`GetResult`方法

```cs
static async Task Test(){
    Task<string> task = Task.Run(() =>{
        Console.WriteLine("另一个线程在运行！");  // 这句话只会被执行一次
        Thread.Sleep(2000);
        return "Hello World";
    });

    // 这里主线程会挂起等待，直到task执行完毕我们拿到返回结果
    var result = task.GetAwaiter().GetResult();
    // 这里不会挂起等待，因为task已经执行完了，我们可以直接拿到结果
    var result2 = await task;
    Console.WriteLine(str);
}
```

#### async和await使用建议

* `async`方法需在其主体中具有`await` 关键字，否则它们将永不暂停。同时C# 编译器将生成一个警告，此代码将会以类似普通方法的方式进行编译和运行。 请注意这会导致效率低下，因为由 C# 编译器为异步方法生成的状态机将不会完成任何任务。
* 应将 `Async` 作为后缀添加到所编写的每个异步方法名称中。这是 .NET 中的惯例，以便更轻松区分同步和异步方法。
* `async void` 应仅用于事件处理程序。因为事件不具有返回类型（因此无法返回 `Task` 和 `Task<T>`）。 其他任何对 `async void` 的使用都不遵循 `TAP` 模型，且可能存在一定使用难度。
* 避免上下文，调用 `ConfigureAwait` 并且传递false不要捕捉当前上下文。

例如：`async void` 方法中引发的异常无法在该方法外部被捕获或十分难以测试 `async void` 方法。

#### async和await总结

`async/await` 本质上只是一个语法糖，它并不产生线程，只是在编译时把语句的执行逻辑改了，相当于过去我们用`callback`，这里编译器自动实现了。

线程的转换是通过`SynchronizationContext`来实现，如果做了`Task.ConfigureAwait(false)`操作，运行`MoveNext`时就只是在线程池中拿个空闲线程出来执行；
如果 `Task.ConfigureAwait(true)`-(默认)，则会在异步操作前`Capture`当前线程的`SynchronizationContext`，异步操作之后运行`MoveNext`时通过`SynchronizationContext`转到目标之前的线程。

一般是想更新UI则需要用到 `SynchronizationContext`，如果异步操作完成还需要做大量运算，则可以考虑`Task.ConfigureAwait(false)`把计算放到后台算，防止UI卡死。

另外还有在异步操作前做的`ExecutionContext.FastCapture`，获取当前线程的执行上下文，注意，如果`Task.ConfigureAwait(false)`，会有个`IgnoreSynctx`的标记，表示在`ExecutionContext.Capture`里不做`SynchronizationContext.Capture`操作，`Capture`到的执行上下文用来在`awaiter completed`后给`MoveNext`用，使`MoveNext`可以有和前面线程同样的上下文。

通过`SynchronizationContext.Post`操作，可以使异步异常在最开始的`try..catch`块中轻松捕获。

### 调用异步方法

在一个异步方法里，可以调用一个或多个异步方法，如何编码取决于异步方法间结果是否相互依赖。

#### 1.顺序调用异步方法

使用`await`关键词可以调用每个异步方法，如果一个异步方法需要使用另一个异步方法的结果，`await`关键词就非常必要。

#### 2.使用组合器

如果异步方法间相互不依赖，则每个异步方法都不使用`await`，而是把每个异步方法的结果赋值给Task变量，就会运行得更快。

* WhenAll是在所有传入的任务都完成时才返回Task。

* WhenAny是在传入的任务其中一个完成就会返回Task。

### 异常处理

#### 单个异步方法处理

异步方法的一个较好异常处理方式，是使用`await`关键字，将其放在`try/catch`中。

返回void的异步方法不会等待。这是因为从`async void`方法抛出的异常无法捕获。因此异步方法最好返回一个Task类型。

```cs
//异步方法错误处理
static async void HandleError()
{
    try
    {
        await ThrowAfter(2000, "HandleError Error");
    }
    catch (Exception ex)
    {

        Console.WriteLine(ex.Message);
    }
}
//在延迟后抛出异常
static async Task ThrowAfter(int ms, string message)
{
    await Task.Delay(ms);
    throw new Exception(message);
}
```

#### 多个异步方法

```cs
// Task.WhenAll，在catch块内可以访问，再使用IsFaulted属性检查任务的状态，
// 以确认它们是否出现错误，然后再进行处理。
static async void HandleError()
{
    Task t1 = null;
    Task t2 = null;
    try
    {
        t1 = ThrowAfter(1000, "HandleError-One-Error");
        t2 = ThrowAfter(2000, "HandleError-Two-Error");
        await Task.WhenAll(t1, t2);
    }
    catch (Exception)
    {
        if (t1.IsFaulted)
            Console.WriteLine(t1.Exception.InnerException.Message);
        if (t2.IsFaulted)
            Console.WriteLine(t2.Exception.InnerException.Message);
    }
}

// 使用AggregateException处理方式
static async void HandleError()
{
    Task taskResult = null;
    try
    {
        Task t1 = ThrowAfter(1000, "HandleError-One-Error");
        Task t2 = ThrowAfter(2000, "HandleError-Two-Error");
        await (taskResult = Task.WhenAll(t1, t2));
    }
    catch (Exception)
    {
        foreach (var ex in taskResult.Exception.InnerExceptions)
        {
            Console.WriteLine(ex.Message);
        }
  
    }
}
//在延迟后抛出异常
static async Task ThrowAfter(int ms, string message)
{
    await Task.Delay(ms);
    throw new Exception(message);
}
```

参考：

[异步编程 In .NET](http://www.cnblogs.com/jesse2013/p/Asynchronous-Programming-In-DotNet.html)

[async/await IL翻译](https://www.cnblogs.com/CrabMan/p/5436083.html)

[async & await 的前世今生（Updated）](http://www.cnblogs.com/jesse2013/p/async-and-await.html)

[异步编程（async&await）](https://www.cnblogs.com/jonins/p/9558275.html)

[深入理解Async/Await](http://www.xyting.org/2017/02/28/understand-async-await-in-depth.html)

[async 和 await 关键字](http://blog.51cto.com/cnn237111/1103029)

[使用Nito.AsyncEx实现异步锁](http://www.cnblogs.com/1zhk/p/5269279.html)

[使用 Async 和 Await 的异步编程 (C#)](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/async/)

[https://docs.microsoft.com/zh-cn/dotnet/csharp/async](https://docs.microsoft.com/zh-cn/dotnet/csharp/async)

[https://docs.microsoft.com/zh-cn/dotnet/standard/asynchronous-programming-patterns/asynchronous-programming-model-apm](https://docs.microsoft.com/zh-cn/dotnet/standard/asynchronous-programming-patterns/asynchronous-programming-model-apm)