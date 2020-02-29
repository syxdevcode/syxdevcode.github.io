---
title: DotNet基础-并行编程之Task
date: 2018-12-22 13:52:20
tags:
- DotNet
- CSharp
- Task
- CSharp基础
- .net异步与并行
categories: 
- .net异步与并行
---
&emsp;&emsp;在.NET 4中的并行编程是依赖Task Parallel Library(后面简称为TPL) 实现的。在TPL中，最基本的执行单元是task(中文可以理解为"任务"),一个task就代表了你要执行的一个操作。你可以为你所要执行的每一个操作定义一个task,TPL就负责创建线程来执行你所定义的task，并且管理线程。TPL是面向task的，自动的；而传统的多线程是以人工为导向的。

&emsp;&emsp;Task机制使得我们把注意力关注在我们要解决的问题上面。如果之前的多线程技术使得我们放弃了一些并行编程的使用，那么.NET 4中的新的并行编程技术可以让我们重新建立信心。虽然有了新的并行技术，但是传统的多线程的技术还是很有用的。任务并行（TPL）是一种范式，也是一组API，可以将大任务分割为多个小任务，并在多个线程中执行。当我们使用TPL中的并行技术的时候来执行多个task的时候，我们不用在关心底层创建线程，管理线程等。

## 线程基础

多核处理器带有一个以上的物理内核，每个物理内核都可能会提供多个硬件线程，也称之为逻辑内核或者逻辑处理器。

Windows将每一个硬件线程识别为一个可调度的逻辑处理器，每一个逻辑处理器可以运行软件线程代码，运行多个软件线程的进程可以充分发挥硬件线程和物理内核的优势，并行地运行指令。Windows会给每一个可用的硬件线程分配一块块的处理时间，并通过这种方式运行上百个千个软件线程。

### 硬件线程

物理内核：计算机实际内核数量；
硬件线程又叫做逻辑内核，可以在”任务管理器“中查看”性能“标签页

一般情况下，一个物理内核对应一个逻辑内核。当然如果你的cpu采用的是超线程技术，那么可能就会有4个物理内核对应。

### 软件线程

传统的代码都是串行的，就一个主线程，当我们为了实现加速而开了很多工作线程，这些工作线程也就是软件线程。

## Task的基本使用

### 创建任务Task

开启task有两种方式:

```cs
static void Main(string[] args)
{
    //第一种方式开启
    var task1 = new Task(() =>
    {
        Run1();
    });

    //第二种方式开启
    var task2 = Task.Factory.StartNew(() =>
    {
        Run2();
    });

    Console.WriteLine("调用start之前****************************\n");

    //调用start之前的“任务状态”
    Console.WriteLine("task1的状态:{0}", task1.Status);

    Console.WriteLine("task2的状态:{0}", task2.Status);

    task1.Start();

    Console.WriteLine("\n调用start之后****************************");

    //调用start之前的“任务状态”
    Console.WriteLine("\ntask1的状态:{0}", task1.Status);

    Console.WriteLine("task2的状态:{0}", task2.Status);

    //主线程等待任务执行完
    Task.WaitAll(task1, task2);

    Console.WriteLine("\n任务执行完后的状态****************************");

    //调用start之前的“任务状态”
    Console.WriteLine("\ntask1的状态:{0}", task1.Status);

    Console.WriteLine("task2的状态:{0}", task2.Status);

    Console.ReadKey();
}

private static void Run1()
{
    Thread.Sleep(1000);
    Console.WriteLine("\n我是任务1");
}

private static void Run2()
{
    Thread.Sleep(2000);
    Console.WriteLine("我是任务2");
}
```

** Task.Run 和 Task.Factory.StartNew 区别**

&emsp;&emsp;可以认为 `Task.Run` 是简化的 `Task.Factory.StartNew` 的使用，除了需要指定一个线程是长时间占用的，否则就使用 `Task.Run`； `Task.Factory.StartNew` 可以设置线程是长时间运行，这时线程池就不会等待这个线程回收。

### 为创建的Task传入参数

想向 `Task` 传入参数，只能用`System.Action<object>`;

```cs
static void Main(string[] args)
{
    string[] messages = { "First task", "Second task", "Third task", "Fourth task" };
    foreach (string msg in messages)
    {
        Task myTask = new Task(obj => printMessage((string)obj), msg);
        myTask.Start();
    }
    // wait for input before exiting
    Console.WriteLine("Main method complete. Press enter to finish.");
    Console.ReadLine();
}

static void printMessage(string message)
{
    Console.WriteLine("Message: {0}", message);
}
```

在创建Task的时候，Task有很多的构造函数的重载，一个主要的重载就是传入TaskCreateOptions的枚举：

* TaskCreateOptions.None:用默认的方式创建一个Task
* TaskCreateOptions.PreferFairness:请求scheduler尽量公平的执行Task(Task和线程一样，有优先级的)
* TaskCreateOptions.LongRunning:声明Task将会长时间的运行。
* TaskCreateOptions.AttachToParent:因为Task是可以嵌套的，所以这个枚举就是把一个子task附加到一个父task中。

&emsp;&emsp;最后要提到的一点就是，我们可以在Task的执行体中用Task.CurrentId来返回Task的唯一表示ID(int)。如果在Task执行体外使用这个属性就会得到null。

### Task执行

#### 等待Task执行完成

用Wait()方法来一直等待一个Task执行完成。当task执行完成，或者被cancel，或者抛出异常，这个方法才会返回。可以使用Wait()方法的不同重载。

```cs
Task task = createTask(token);
task.Start();

Console.WriteLine("Waiting for task to complete.");
task.Wait();
Console.WriteLine("Task Completed.");

// create and start another task
task = createTask(token);
task.Start();
Console.WriteLine("Waiting 2 secs for task to complete.");
bool completed = task.Wait(6000);
Console.WriteLine("Wait ended - task completed: {0}", completed);

// create and start another task
task = createTask(token);
task.Start();
Console.WriteLine("Waiting 2 secs for task to complete.");
completed = task.Wait(2000, token);
Console.WriteLine("Wait ended - task completed: {0} task cancelled {1}",
completed, task.IsCanceled);
```

<font color=#0099ff size=4 face="黑体">wait方法子task执行完成之后会返回true。</font>

#### 等待多个task

```cs
Task.WaitAll(task1, task2);
```

#### 等待多个task中的一个task执行完成

```cs
int taskIndex = Task.WaitAny(task1, task2);
```

### 懒加载的Task(Lazily Task)

延迟初始化，主要的好处就是避免不必要的系统开销。

Lazy变量只有在用到的时候才会被初始化。所以我们可以把Lazy变量和task的创建结合：只有这个task要被执行的时候才去初始化。现在如果用了Lazy的task，那么现在我们初始化的就是那个Lazy变量了，而没有初始化task，(初始化Lazy变量的开销小于初始化task)，只有当调用了lazyData.Value时，Lazy变量中包含的那个task才会初始化。

```cs
static void Main(string[] args)
{
    // define the function
    Func<string> taskBody = new Func<string>(() =>
    {
        Console.WriteLine("Task body working...");
        return "Task Result";
    });

    // create the lazy variable
    Lazy<Task<string>> lazyData = new Lazy<Task<string>>(() =>
    Task<string>.Factory.StartNew(taskBody));

    Console.WriteLine("Calling lazy variable");
    Console.WriteLine("Result from task: {0}", lazyData.Value.Result);

    // do the same thing in a single statement
    Lazy<Task<string>> lazyData2 = new Lazy<Task<string>>(
    () => Task<string>.Factory.StartNew(() =>
    {
        Console.WriteLine("Task body working...");
        return "Task Result";
    }));

    Console.WriteLine("Calling second lazy variable");
    Console.WriteLine("Result from task: {0}", lazyData2.Value.Result);

    // wait for input before exiting
    Console.WriteLine("Main method complete. Press enter to finish.");
    Console.ReadLine();
}
```

### Task 死锁

描述：如果有两个或者多个task(简称TaskA)等待其他的task（TaskB）执行完成才开始执行，但是TaskB也在等待TaskA执行完成才开始执行，这样死锁就产生了。

解决方案：避免这个问题最好的方法就是：不要使的task来依赖其他的task。也就是说，最好不要在你定义的task的执行体内包含其他的task。

### Task异常处理

在执行 `Task.Wait()`,`Task.WaitAll()`,`Task.WaitAny()`,`Task.Result`.不管那里出现了异常，最后抛出的就是一个 `System.AggregateException`。

#### 处理基本的异常

```cs
// create the tasks
Task task1 = new Task(() =>
{
    ArgumentOutOfRangeException exception = new ArgumentOutOfRangeException();
    exception.Source = "task1";
    throw exception;
});
Task task2 = new Task(() =>
{
    throw new NullReferenceException();
});
Task task3 = new Task(() =>
{
    Console.WriteLine("Hello from Task 3");
});
// start the tasks
task1.Start(); task2.Start(); task3.Start();
// wait for all of the tasks to complete
// and wrap the method in a try...catch block
try
{
    Task.WaitAll(task1, task2, task3);
}
catch (AggregateException ex)
{
    // enumerate the exceptions that have been aggregated
    foreach (Exception inner in ex.InnerExceptions)
    {
        Console.WriteLine("Exception type {0} from {1}",
        inner.GetType(), inner.Source);
    }
}
// wait for input before exiting
Console.WriteLine("Main method complete. Press enter to finish.");
Console.ReadLine();
```

#### 使用迭代的异常处理Handler

一般情况下，我们需要区分哪些异常需要处理，而哪些异常需要继续往上传递。`AggregateException`类提供了一个`Handle()`方法，我们可以用这个方法来处理

AggregateException中的每一个异常。在这个 `Handle()` 方法中，返回true就表明，这个异常我们已经处理了，不用抛出，反之。

```cs
// create the cancellation token source and the token
CancellationTokenSource tokenSource = new CancellationTokenSource();
CancellationToken token = tokenSource.Token;
// create a task that waits on the cancellation token
Task task1 = new Task(() =>
{
    // wait forever or until the token is cancelled
    token.WaitHandle.WaitOne(-1);
    // throw an exception to acknowledge the cancellation
    throw new OperationCanceledException(token);
}, token);
// create a task that throws an exception
Task task2 = new Task(() =>
{
    throw new NullReferenceException();
});
// start the tasks
task1.Start(); task2.Start();
// cancel the token
tokenSource.Cancel();
// wait on the tasks and catch any exceptions
try
{
    Task.WaitAll(task1, task2);
}
catch (AggregateException ex)
{
    // iterate through the inner exceptions using
    // the handle method
    ex.Handle((inner) =>
    {
        if (inner is OperationCanceledException)
        {
            // ...handle task cancellation...
            return true;
        }
        else
        {
            // this is an exception we don't know how
            // to handle, so return false
            return false;
        }
    });
}
// wait for input before exiting
Console.WriteLine("Main method complete. Press enter to finish.");
Console.ReadKey();
```

### 获取Task的执行结果

```cs
private static void Main(string[] args)
{
    // create the task
    Task<int> task1 = new Task<int>(() =>
    {
        int sum = 0;
        for (int i = 0; i < 100; i++)
        {
            sum += i;
        }
        return sum;
    });

    task1.Start();
    // write out the result
    Console.WriteLine("Result 1: {0}", task1.Result);

    // create the task
    Task<int> task2 = Task.Factory.StartNew<int>(() =>
    {
        int sum = 0;
        for (int i = 0; i < 100; i++)
        {
            sum += i;
        }
        return sum;
    });

    // write out the result
    Console.WriteLine("Result 1: {0}", task2.Result);

    Console.ReadLine();
}
```

### 获取Task的状态

在.NET并行编程还有一个已经标准化的操作就是可以获取task的状态，通过 `Task.Status` 属性来得到的，这个属性返回一个 `System.Threading.Tasks.TaskStatus` 的枚举值。

如下：

* Created:表明task已经被初始化了，但是还没有加入到Scheduler中。
* WatingForActivation：task正在等待被加入到Scheduler中。
* WaitingToRun:已经被加入到了Scheduler，等待执行。
* Running：task正在运行
* WaitingForChildrenToComplete:表明父task正在等待子task运行结束。
* RanToCompletion:表明task已经执行完了，但是还没有被cancel，而且也这个task也没有抛出异常。
* Canceled:表明task已经被cancel了。（大家可以参看之前讲述取消task的文章）
* Faulted:表明task在运行的时候已经抛出了异常。

### 取消任务（Task）

#### 通过轮询的方式检测Task是否被取消

&emsp;&emsp;在很多Task内部都包含了循环，用来处理数据。我们可以在循环中通过 `CancellationToken` 的`IsCancellationRequest` 属性来检测task是否被取消了。如果这个属性为true，那么我们就得跳出循环，并且释放task所占用的资源(如数据库资源，文件资源等)。

```cs
static void Main(string[] args)
{
    // create the cancellation token source
    CancellationTokenSource tokenSource = new CancellationTokenSource();

    // create the cancellation token
    CancellationToken token = tokenSource.Token;
    // create the task

    Task task = new Task(() =>
    {
        for (int i = 0; i < int.MaxValue; i++)
        {
            if (token.IsCancellationRequested)
            {
                Console.WriteLine("Task cancel detected");
                throw new OperationCanceledException(token);
            }
            else
            {
                Console.WriteLine("Int value {0}", i);
            }
        }
    }, token);

    // wait for input before we start the task
    Console.WriteLine("Press enter to start task");
    Console.WriteLine("Press enter again to cancel task");
    Console.ReadLine();

    // start the task
    task.Start();

    // read a line from the console.
    Console.ReadLine();

    // cancel the task
    Console.WriteLine("Cancelling task");
    tokenSource.Cancel();

    // wait for input before exiting
    Console.WriteLine("Main method complete. Press enter to finish.");
    Console.ReadLine();
}
```

#### 用委托delegate来检测Task是否被取消

```cs
static void Main(string[] args)
{
    // create the cancellation token source
    CancellationTokenSource tokenSource = new CancellationTokenSource();

    // create the cancellation token
    CancellationToken token = tokenSource.Token;

    // create the task
    Task task = new Task(() =>
    {
        for (int i = 0; i < int.MaxValue; i++)
        {
            if (token.IsCancellationRequested)
            {
                Console.WriteLine("Task cancel detected");
                throw new OperationCanceledException(token);
            }
            else
            {
                Console.WriteLine("Int value {0}", i);
            }
        }
    }, token);

    // register a cancellation delegate
    token.Register(() =>
    {
        Console.WriteLine(">>>>>> Delegate Invoked\n");
    });

    // wait for input before we start the task
    Console.WriteLine("Press enter to start task");
    Console.WriteLine("Press enter again to cancel task");
    Console.ReadLine();

    // start the task
    task.Start();
    // read a line from the console.
    Console.ReadLine();

    // cancel the task
    Console.WriteLine("Cancelling task");
    tokenSource.Cancel();

    // wait for input before exiting
    Console.WriteLine("Main method complete. Press enter to finish.");
    Console.ReadLine();
}
```

#### 用Wait Handle还检测Task是否被取消

&emsp;&emsp;检测task是否被cancel就是调用CancellationToken.WaitHandle属性。对于这个属性的详细使用，在后续的文章中会深入的讲述，在这里主要知道一点就行了：CancellationToken的WaitOne()方法会阻止task的运行，只有CancellationToken的cancel()方法被调用后，这种阻止才会释放。

&emsp;&emsp;在下面的例子中，创建了两个task，其中task2调用了WaitOne()方法，所以task2一直不会运行，除非调用了CancellationToken的Cancel()方法，所以WaitOne()方法也算是检测task是否被cancel的一种方法了。

```cs
static void Main(string[] args)
{

    // create the cancellation token source
    CancellationTokenSource tokenSource = new CancellationTokenSource();

    // create the cancellation token
    CancellationToken token = tokenSource.Token;

    // create the task
    Task task1 = new Task(() =>
    {
        for (int i = 0; i < int.MaxValue; i++)
        {
            if (token.IsCancellationRequested)
            {
                Console.WriteLine("Task cancel detected");
                throw new OperationCanceledException(token);
            }
            else
            {
                Console.WriteLine("Int value {0}", i);
            }
        }
    }, token);

    // create a second task that will use the wait handle
    Task task2 = new Task(() =>
    {
        // wait on the handle
        token.WaitHandle.WaitOne();
        // write out a message
        Console.WriteLine(">>>>> Wait handle released");
    });

    // wait for input before we start the task
    Console.WriteLine("Press enter to start task");
    Console.WriteLine("Press enter again to cancel task");
    Console.ReadLine();
    // start the tasks
    task1.Start();
    task2.Start();

    // read a line from the console.
    Console.ReadLine();

    // cancel the task
    Console.WriteLine("Cancelling task");
    tokenSource.Cancel();

    // wait for input before exiting
    Console.WriteLine("Main method complete. Press enter to finish.");
    Console.ReadLine();
}
```

#### 取消多个Task

&emsp;&emsp;我们可以使用一个CancellationToken来创建多个不同的Tasks，当这个CancellationToken的Cancel()方法调用的时候，使用了这个token的多个task都会被取消。

#### 创建组合的取消Task的Token

&emsp;&emsp;我们可以用CancellationTokenSource.CreateLinkedTokenSource()方法来创建一个组合的token，这个组合的token有很多的CancellationToken组成。主要组合token中的任意一个token调用了Cancel()方法，那么使用这个组合token的所有task就会被取消。

```cs
static void Main(string[] args)
{
    // create the cancellation token sources
    CancellationTokenSource tokenSource1 = new CancellationTokenSource();
    CancellationTokenSource tokenSource2 = new CancellationTokenSource();
    CancellationTokenSource tokenSource3 = new CancellationTokenSource();

    // create a composite token source using multiple tokens
    CancellationTokenSource compositeSource =
        CancellationTokenSource.CreateLinkedTokenSource(
    tokenSource1.Token, tokenSource2.Token, tokenSource3.Token);

    // create a cancellable task using the composite token
    Task task = new Task(() =>
    {
        // wait until the token has been cancelled
        compositeSource.Token.WaitHandle.WaitOne();
        // throw a cancellation exception
        throw new OperationCanceledException(compositeSource.Token);
    }, compositeSource.Token);

    // start the task
    task.Start();

    // cancel one of the original tokens
    tokenSource2.Cancel();

    // wait for input before exiting
    Console.WriteLine("Main method complete. Press enter to finish.");
    Console.ReadLine();
}
```

#### 判断一个Task是否已经被取消了

可以使用Task的IsCancelled属性来判断task是否被取消了。

### Task的休眠

#### 使用CancellationToken的Wait Handle

```cs
static void Main(string[] args)
{
    // create the cancellation token source
    CancellationTokenSource tokenSource = new CancellationTokenSource();

    // create the cancellation token
    CancellationToken token = tokenSource.Token;

    // create the first task, which we will let run fully
    Task task1 = new Task(() =>
    {
        for (int i = 0; i < Int32.MaxValue; i++)
        {
            // put the task to sleep for 10 seconds
            bool cancelled = token.WaitHandle.WaitOne(10000);
            // print out a message
            Console.WriteLine("Task 1 - Int value {0}. Cancelled? {1}",
            i, cancelled);
            // check to see if we have been cancelled
            if (cancelled)
            {
                throw new OperationCanceledException(token);
            }
        }
    }, token);
    // start task
    task1.Start();

    // wait for input before exiting
    Console.WriteLine("Press enter to cancel token.");
    Console.ReadLine();

    // cancel the token
    tokenSource.Cancel();

    // wait for input before exiting
    Console.WriteLine("Main method complete. Press enter to finish.");
    Console.ReadLine();
}
```

&emsp;&emsp;有一点要注意：WaitOne()方法只有在设定的时间间隔到了，或者Cancel方法被调用，此时task才会被唤醒。如果如果cancel()方法被调用而导致task被唤醒，那么CancellationToken.WaitHandle.WaitOne()方法就会返回true，如果是因为设定的时间到了而导致task唤醒，那么CancellationToken.WaitHandle.WaitOne()方法返回false。

#### Task.Delay

Task.Delay在不阻塞当前线程的情况下使用逻辑延迟时使用。Task.Delay旨在异步运行。

#### 使用传统的Sleep

Thread.Sleep时要阻止当前线程。

注意：Thread.Sleep在同步代码中使用，不能在异步代码中使用；

`Thread.Sleep(10000);`

&emsp;&emsp;使用Thread.Sleep()之后，然后再调用token的cancel方法，task不会立即就被cancel，这主要是因为Thread.Sleep()将会一直阻塞线程，直到达到了设定的时间，这之后，再去check task时候被cancel了。举个例子，假设再task方法体内调用Thread.Sleep(100000)方法来休眠task，然后再后面的代码中调用token.Cancel()方法，此时处于并行编程内部机制不会去检测task是否已经发出了cancel请求，而是一直休眠，直到时间超过了100000微秒。如果采用的是之前的第一种休眠方法，那么不管WaitOne（）中设置了多长的时间，只要token.Cancel()被调用，那么task就像内部的Scheduler发出了cancel的请求，而且task会被cancel。

#### 自旋等待

推荐的方法。

之前的两种方法，当他们使得task休眠的时候，这些task已经从Scheduler的管理中退出来了，不被再内部的Scheduler管理(Scheduler是负责管理线程的)，因为休眠的task已经不被Scheduler管理了，所以Scheduler必须做一些工作去决定下一步是哪个线程要运行，并且启动它。为了避免Scheduler做那些工作，我们可以采用自旋等待：此时这个休眠的task所对应的线程不会从Scheduler中退出，这个task会把自己和CPU的轮转关联起来。

```cs
static void Main(string[] args)
{
    // create the cancellation token source
    CancellationTokenSource tokenSource = new CancellationTokenSource();

    // create the cancellation token
    CancellationToken token = tokenSource.Token;

    // create the first task, which we will let run fully
    Task task1 = new Task(() =>
    {
        for (int i = 0; i < Int32.MaxValue; i++)
        {
            // put the task to sleep for 10 seconds
            Thread.SpinWait(10000);
            // print out a message
            Console.WriteLine("Task 1 - Int value {0}", i);
            // check for task cancellation
            token.ThrowIfCancellationRequested();
        }
    }, token);

    // start task
    task1.Start();

    // wait for input before exiting
    Console.WriteLine("Press enter to cancel token.");

    Console.ReadLine();
    // cancel the token
    tokenSource.Cancel();

    // wait for input before exiting
    Console.WriteLine("Main method complete. Press enter to finish.");
    Console.ReadLine();
}
```

&emsp;&emsp;代码中我们在Thread.SpinWait()方法中传入一个整数，这个整数就表示CPU时间片轮转的次数，至于要等待多长的时间，这个就和计算机有关了，不同的计算机，CPU的轮转时间不一样。自旋等待的方法常常于获得同步锁，后续会讲解。使用自旋等待会一直占用CPU，而且也会消耗CPU的资源，更大的问题就是这个方法会影响Scheduler的运作。

参考：

[.NET 4 并行(多核)编程系列](https://www.cnblogs.com/yanyangtian/category/246954.html)

[8天玩转并行开发——第二天 Task的使用](https://www.cnblogs.com/huangxincheng/archive/2012/04/03/2430638.html)

[C# Task.Run 和 Task.Factory.StartNew 区别](https://lindexi.github.io/lindexi/post/C-Task.Run-%E5%92%8C-Task.Factory.StartNew-%E5%8C%BA%E5%88%AB.html)