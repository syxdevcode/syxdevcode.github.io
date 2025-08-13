---
title: TaskCompletionSource使用
date: 2018-12-24 15:35:43
tags:
- DotNet
- CSharp
- Task
- .net异步与并行
categories: 
- .net异步与并行
---
在目标方法要调用基于事件API，又要返回Task的时候使用。

比如下面的ApiWrapper方法，该方法要返回Task<string>,又要调用EventClass对象的Do方法，并且等到Do方法触发Done事件后，Task才能得到结果并返回。

```cs
private void test()
{
    var task = ApiWrapper();

    Console.WriteLine("Test CurrentThread：" + Thread.CurrentThread.ManagedThreadId);

    Console.WriteLine(task.Result);
}

private static Task<string> ApiWrapper()
{
    var tcs = new TaskCompletionSource<string>();

    var api = new EventClass();
    api.Done += (args) => { tcs.TrySetResult(args); };
    api.Do();

    return tcs.Task;
}

public class EventClass
{
    public Action<string> Done = (args) => {; };

    public void Do()
    {
        Console.WriteLine("EventClass" + Thread.CurrentThread.ManagedThreadId);
        Done("Done");
    }
}
```

参考：

[TaskCompletionSource<TResult> Class](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.tasks.taskcompletionsource-1?view=netframework-4.7.2)

[什么时候应该使用TaskCompletionSource<T>](https://cloud.tencent.com/developer/ask/72340)

[TaskCompletionSource的使用场景](https://www.cnblogs.com/1zhk/p/5399538.html)