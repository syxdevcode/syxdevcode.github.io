---
title: 并行开发之Barrier
date: 2019-03-06 08:44:48
tags:
- DotNet
- CSharp
- Task
- .net异步与并行
categories: 
- .net异步与并行
---
## 屏障简介

`System.Threading.Barrier` 是同步基元，可以使多个线程（称为“参与者”）分阶段同时处理算法。
屏障表示工作阶段的末尾。 单个参与者到达屏障后将被阻止，直至所有参与者都已达到同一障碍。 所有参与者都已达到屏障后，你可以选择调用阶段后操作。此阶段后操作可由单线程用于执行操作，而所有其他线程仍被阻止。 执行此操作后，所有参与者将不受阻止。

## 添加和删除参与者

创建 `Barrier` 实例时，需指定参与者数量。 还可以随时动态添加或删除参与者。 例如，如果其中一个参与者解决了问题的一部分，可以存储结果，停止执行相应线程，并调用 `Barrier.RemoveParticipant` 以减少屏障中的参与者数量。 当通过调用 `Barrier.AddParticipant` 添加参与者时，返回值将指定当前阶段的数量，这在初始化新的参与者的工作时很有用。

## 断开的屏障

如果一个参与者无法到达屏障，则可能发生死锁。若要避免这些死锁，请使用 Barrier.SignalAndWait 方法的重载来指定超时期限和取消标记。 这些重载将返回一个布尔值，每个参与者均可在继续到下一阶段前进行检查。

```cs
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp4
{
    internal class Program
    {
        //四个task执行
        private static Task[] tasks = new Task[4];

        private static Barrier _barrier = null;

        private static void Main(string[] args)
        {
            CancellationTokenSource cts = new CancellationTokenSource();

            CancellationToken ct = cts.Token;

            _barrier = new Barrier(tasks.Length, (i) =>
            {
                Console.WriteLine("**********************************************************");
                Console.WriteLine("\n屏障中当前阶段编号:{0}\n", i.CurrentPhaseNumber);
                Console.WriteLine("**********************************************************");
            });

            for (int j = 0; j < tasks.Length; j++)
            {
                tasks[j] = Task.Factory.StartNew((obj) =>
                {
                    var single = Convert.ToInt32(obj);

                    LoadUser(single);

                    if (!_barrier.SignalAndWait(2000))
                    {
                        //抛出异常，取消后面加载的执行
                        throw new OperationCanceledException(string.Format("我是当前任务{0},我抛出异常了！", single), ct);
                    }

                    LoadProduct(single);
                    _barrier.SignalAndWait();

                    LoadOrder(single);
                    _barrier.SignalAndWait();
                }, j, ct);
            }

            //等待所有tasks 4s
            Task.WaitAll(tasks, 4000);

            try
            {
                for (int i = 0; i < tasks.Length; i++)
                {
                    if (tasks[i].Status == TaskStatus.Faulted)
                    {
                        //获取task中的异常
                        foreach (var single in tasks[i].Exception.InnerExceptions)
                        {
                            Console.WriteLine(single.Message);
                        }
                    }
                }

                _barrier.Dispose();
            }
            catch (AggregateException e)
            {
                Console.WriteLine("我是总异常:{0}", e.Message);
            }

            Console.Read();
        }

        private static void LoadUser(int num)
        {
            Console.WriteLine("\n当前任务:{0}正在加载User部分数据！", num);

            if (num == 0)
            {
                //自旋转5s
                if (!SpinWait.SpinUntil(() => _barrier.ParticipantsRemaining == 0, 5000))
                    return;
            }

            Console.WriteLine("当前任务:{0}正在加载User数据完毕！", num);
        }

        private static void LoadProduct(int num)
        {
            Console.WriteLine("当前任务:{0}正在加载Product部分数据！", num);
        }

        private static void LoadOrder(int num)
        {
            Console.WriteLine("当前任务:{0}正在加载Order部分数据！", num);
        }
    }
}
```

## 阶段后异常

如果阶段后委托引发异常，则它将包装在 BarrierPostPhaseException 对象中，然后传播到所有参与者。

## 屏障与 ContinueWhenAll

<font color=#ff0000 size=4 face="黑体">当线程执行循环中的多个阶段时，屏障特别有用。</font> 如果你的代码仅需一个或多个工作阶段，则应考虑是否配合使用 `System.Threading.Tasks.Task` 对象与任何类型的隐式联接，其中包括：

* TaskFactory.ContinueWhenAll

* Parallel.Invoke

* Parallel.ForEach

* Parallel.For

## 实例代码

msdn官方实例：
统计两个线程使用随机算法重新随机选择字词，分别在同一阶段查找一半解决方案时所需的迭代次数（或阶段数）。

```cs
using System;
using System.Text;
using System.Threading;

namespace ConsoleApp4
{
    internal class Program
    {
        private static string[] words1 = new string[] { "brown", "jumps", "the", "fox", "quick" };
        private static string[] words2 = new string[] { "dog", "lazy", "the", "over" };
        private static string solution = "the quick brown fox jumps over the lazy dog.";

        private static bool success = false;

        private static Barrier barrier = new Barrier(2, (b) =>
        {
            StringBuilder sb = new StringBuilder();
            for (int i = 0; i < words1.Length; i++)
            {
                sb.Append(words1[i]);
                sb.Append(" ");
            }
            for (int i = 0; i < words2.Length; i++)
            {
                sb.Append(words2[i]);

                if (i < words2.Length - 1)
                    sb.Append(" ");
            }
            sb.Append(".");

            Console.CursorLeft = 0;
            Console.Write("Current phase: {0}", barrier.CurrentPhaseNumber);
            if (String.CompareOrdinal(solution, sb.ToString()) == 0)
            {
                success = true;
                Console.WriteLine("\r\nThe solution was found in {0} attempts", barrier.CurrentPhaseNumber);
            }
        });

        private static void Main(string[] args)
        {
            Thread t1 = new Thread(() => Solve(words1));
            Thread t2 = new Thread(() => Solve(words2));
            t1.Start();
            t2.Start();

            // Keep the console window open.
            Console.ReadLine();
        }

        private static void Solve(string[] wordArray)
        {
            while (success == false)
            {
                Random random = new Random();
                for (int i = wordArray.Length - 1; i > 0; i--)
                {
                    int swapIndex = random.Next(i + 1);
                    string temp = wordArray[i];
                    wordArray[i] = wordArray[swapIndex];
                    wordArray[swapIndex] = temp;
                }

                barrier.SignalAndWait();
            }
        }
    }
}
```

简单示例：

```cs
using System;
using System.Threading;
using System.Threading.Tasks;

namespace ConsoleApp4
{
    internal class Program
    {
        //四个task执行
        private static Task[] tasks = new Task[4];

        private static Barrier barrier = null;

        private static void Main(string[] args)
        {
            barrier = new Barrier(tasks.Length, (i) =>
            {
                Console.WriteLine("**********************************************************");
                Console.WriteLine("\n屏障中当前阶段编号:{0}\n", i.CurrentPhaseNumber);
                Console.WriteLine("**********************************************************");
            });

            for (int j = 0; j < tasks.Length; j++)
            {
                tasks[j] = Task.Factory.StartNew((obj) =>
                {
                    var single = Convert.ToInt32(obj);

                    LoadUser(single);
                    barrier.SignalAndWait();

                    LoadProduct(single);
                    barrier.SignalAndWait();

                    LoadOrder(single);
                    barrier.SignalAndWait();
                }, j);
            }

            Task.WaitAll(tasks);

            Console.WriteLine("指定数据库中所有数据已经加载完毕！");

            Console.Read();
        }

        private static void LoadUser(int num)
        {
            Console.WriteLine("当前任务:{0}正在加载User部分数据！", num);

            // 以下代码会造成死锁
            /*
            if (num == 0)
            {
                //num=0：表示0号任务
                //barrier.ParticipantsRemaining == 0：表示所有task到达屏障才会退出
                // SpinWait.SpinUntil: 自旋锁，相当于死循环
                SpinWait.SpinUntil(() => barrier.ParticipantsRemaining == 0);
            }*/
        }

        private static void LoadProduct(int num)
        {
            Console.WriteLine("当前任务:{0}正在加载Product部分数据！", num);
        }

        private static void LoadOrder(int num)
        {
            Console.WriteLine("当前任务:{0}正在加载Order部分数据！", num);
        }
    }
}
```

参考：

[屏障](https://docs.microsoft.com/zh-cn/dotnet/standard/threading/barrier)

[如何：使用屏障来使并发操作保持同步](https://docs.microsoft.com/zh-cn/dotnet/standard/threading/how-to-synchronize-concurrent-operations-with-a-barrier)

[8天玩转并行开发——第四天 同步机制（上）](https://www.cnblogs.com/huangxincheng/archive/2012/04/07/2437110.html)