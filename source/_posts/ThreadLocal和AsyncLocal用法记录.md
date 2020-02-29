---
title: ThreadLocal和AsyncLocal用法记录
date: 2019-09-27 15:38:41
tags:
- DotNet
- CSharp
- .net异步与并行
categories: 
- .net异步与并行
---
## AsyncLocal

表示对于给定异步控制流（如异步方法）是本地数据的环境数据。

注意：<font color=#ff0000 size=4 face="黑体">await外的AsyncLocal值可以传递到await内, await内的AsyncLocal值无法传递到await外(只能读取不能修改)。</font>

例子：

```cs
namespace ConsoleApp7
{
    class Program
    {
        static void Main(string[] args)
        {
            var task = Task.Run(async () =>
            {
                v.Value = 123;
                await Intercept.Invoke(); // value = 888

                // 恢复备份的上下文
                Console.WriteLine(Program.v.Value); // 输出 123
            });
            task.Wait();

            Console.ReadKey();
        }

       public static AsyncLocal<int> v = new AsyncLocal<int>();
    }

    public class Intercept
    {
        public static async Task Invoke()
        {
            await Task.FromResult(0);

            Program.v.Value = 888;
        }
    }
}
```

msdn例子：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;

class Example
{
    static AsyncLocal<string> _asyncLocalString = new AsyncLocal<string>();

    static ThreadLocal<string> _threadLocalString = new ThreadLocal<string>();

    static async Task AsyncMethodA()
    {
        // Start multiple async method calls, with different AsyncLocal values.
        // We also set ThreadLocal values, to demonstrate how the two mechanisms differ.
        _asyncLocalString.Value = "Value 1";
        _threadLocalString.Value = "Value 1";
        var t1 = AsyncMethodB("Value 1");

        _asyncLocalString.Value = "Value 2";
        _threadLocalString.Value = "Value 2";
        var t2 = AsyncMethodB("Value 2");

        // Await both calls
        await t1;
        await t2;
     }

    static async Task AsyncMethodB(string expectedValue)
    {
        Console.WriteLine("Entering AsyncMethodB.");
        Console.WriteLine("   Expected '{0}', AsyncLocal value is '{1}', ThreadLocal value is '{2}'", 
                          expectedValue, _asyncLocalString.Value, _threadLocalString.Value);
        await Task.Delay(100);
        Console.WriteLine("Exiting AsyncMethodB.");
        Console.WriteLine("   Expected '{0}', got '{1}', ThreadLocal value is '{2}'", 
                          expectedValue, _asyncLocalString.Value, _threadLocalString.Value);
    }

    static async Task Main(string[] args)
    {
        await AsyncMethodA();
    }
}
// The example displays the following output:
//   Entering AsyncMethodB.
//      Expected 'Value 1', AsyncLocal value is 'Value 1', ThreadLocal value is 'Value 1'
//   Entering AsyncMethodB.
//      Expected 'Value 2', AsyncLocal value is 'Value 2', ThreadLocal value is 'Value 2'
//   Exiting AsyncMethodB.
//      Expected 'Value 2', got 'Value 2', ThreadLocal value is ''
//   Exiting AsyncMethodB.
//      Expected 'Value 1', got 'Value 1', ThreadLocal value is ''
```

## ThreadLocal

提供数据的线程本地存储。

```cs
using System;
using System.Threading;
using System.Threading.Tasks;

class ThreadLocalDemo
{
    // Demonstrates:
    //      ThreadLocal(T) constructor
    //      ThreadLocal(T).Value
    //      One usage of ThreadLocal(T)
    static void Main()
    {
        // Thread-Local variable that yields a name for a thread
        ThreadLocal<string> ThreadName = new ThreadLocal<string>(() =>
        {
            return "Thread" + Thread.CurrentThread.ManagedThreadId;
        });

        // Action that prints out ThreadName for the current thread
        Action action = () =>
        {
            // If ThreadName.IsValueCreated is true, it means that we are not the
            // first action to run on this thread.
            bool repeat = ThreadName.IsValueCreated;

            Console.WriteLine("ThreadName = {0} {1}", ThreadName.Value, repeat ? "(repeat)" : "");
        };

        // Launch eight of them.  On 4 cores or less, you should see some repeat ThreadNames
        Parallel.Invoke(action, action, action, action, action, action, action, action);

        // Dispose when you are done
        ThreadName.Dispose();
    }
}
// This multithreading example can produce different outputs for each 'action' invocation and will vary with each run.
// Therefore, the example output will resemble but may not exactly match the following output (from a 4 core processor):
// ThreadName = Thread5 
// ThreadName = Thread6 
// ThreadName = Thread4 
// ThreadName = Thread6 (repeat)
// ThreadName = Thread1 
// ThreadName = Thread4 (repeat)
// ThreadName = Thread7 
// ThreadName = Thread5 (repeat)
```

参考：

[AsyncLocal<T>](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.asynclocal-1?view=netframework-4.8)

[ThreadLocal<T>](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.threadlocal-1?view=netframework-4.8)

[AsyncLocal的运作机制和陷阱](https://www.cnblogs.com/zkweb/p/7747162.html)

[How to make AsyncLocal flow to siblings?](https://stackoverflow.com/questions/37306203/how-to-make-asynclocal-flow-to-siblings)