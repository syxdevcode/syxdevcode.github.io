---
title: 并行开发之自旋锁-spinLock
date: 2019-03-06 10:58:13
tags:
- DotNet
- CSharp
- Task
- .net异步与并行
categories: 
- .net异步与并行
description: 提供一个相互排斥锁基元，在该基元中，尝试获取锁的线程将在重复检查的循环中等待，直至该锁变为可用为止。
---
## SpinLock Struct

提供一个相互排斥锁基元，在该基元中，尝试获取锁的线程将在重复检查的循环中等待，直至该锁变为可用为止。

```cs
static void SpinLockSample1()
{
    SpinLock sl = new SpinLock();

    StringBuilder sb = new StringBuilder();

    // Action taken by each parallel job.
    // Append to the StringBuilder 10000 times, protecting
    // access to sb with a SpinLock.
    Action action = () =>
    {
        bool gotLock = false;
        for (int i = 0; i < 10000; i++)
        {
            gotLock = false;
            try
            {
                sl.Enter(ref gotLock);
                sb.Append((i % 10).ToString());
            }
            finally
            {
                // Only give up the lock if you actually acquired it
                if (gotLock) sl.Exit();
            }
        }
    };

    // Invoke 3 concurrent instances of the action above
    Parallel.Invoke(action, action, action);

    // Check/Show the results
    Console.WriteLine("sb.Length = {0} (should be 30000)", sb.Length);
    Console.WriteLine("number of occurrences of '5' in sb: {0} (should be 3000)",
        sb.ToString().Where(c => (c == '5')).Count());
}
```

参考：

[SpinLock Struct](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.spinlock?view=netframework-4.7.2)