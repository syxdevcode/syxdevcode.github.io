---
title: Polly基本使用范例
date: 2018-07-26 16:11:28
tags:
- DotNet
- CSharp
- DotNetCore
categories: 
- Polly
---
# 介绍

Polly是一个被.NET基金会认可的弹性和瞬态故障处理库，允许我们以非常顺畅和线程安全的方式来执诸如行重试

官网：[http://www.thepollyproject.org/](http://www.thepollyproject.org/)

源码：[https://github.com/App-vNext/Polly](https://github.com/App-vNext/Polly)

其主要功能如下：

* 重试（Retry）
* 断路器（Circuit-Breaker）
* 超时检测（Timeout）
* 隔板隔离 (Bulkhead Isolation)
* 缓存（Cache）
* 降级（Fallback）

在Polly中，对这些服务容错模式分为两类：

* 错误处理(fault handling)：重试、熔断、回退（降级）
* 弹性应变(resilience)：超时、舱壁、缓存

可以说错误处理是当错误已经发生时，防止由于该错误对整个系统造成更坏的影响而设置。而弹性应变，则在是错误发生前，针对有可能发生错误的地方进行预先处理，从而达到保护整个系统的目地。

# 错误处理(fault handling)

## 重试

```csharp
static void Test1()
{
    try
    {
        ISyncPolicy policy = Policy.Handle<ArgumentException>()
        .Retry(2, (ex, retryCount, context) =>
         {
             Console.WriteLine($"Error occured,runing fallback,exception :{ex.Message},retryCount:{retryCount}");
         });

        policy.Execute(() =>
        {
            Console.WriteLine("Job Start");

            throw new ArgumentException("Hello Polly!");
        });
    }
    catch (Exception ex)
    {
        Console.WriteLine("There's one unhandled exception : " + ex.Message);
    }
}
```

* .Retry(3) ：按次数重试
* .RetryForever() ：不断重试(直到成功)
* .WaitAndRetry() ：等待之后重试
* .WaitAndRetryForever ：等待并永远重试（直到成功

## 熔断 Circuit Breaker

```csharp
static void CircuitBreaker()
{
    Action<Exception, TimeSpan, Context> onBreak = (exception, timespan, context) =>
    {
        Console.WriteLine("onBreak Running : " + exception.Message);
    };

    Action<Context> onReset = context =>
    {
        Console.WriteLine("Job Back to normal");
    };

    CircuitBreakerPolicy breaker = Policy
        .Handle<AggregateException>()
        .CircuitBreaker(3, TimeSpan.FromSeconds(10), onBreak, onReset);

    // Monitor the circuit state, for example for health reporting.
    CircuitState state = breaker.CircuitState;

    ISyncPolicy policy = Policy.Handle<ArgumentException>()
        .Retry(3, (ex, retryCount, context) =>
        {
            Console.WriteLine($"Runing fallback,Exception :{ex.Message},RetryCount:{retryCount}");
        });

    while (true)
    {
        try
        {
            var policyWrap = Policy.Wrap(policy, breaker);

            // (wraps the policies around any executed delegate: fallback outermost ... bulkhead innermost)
            policyWrap.Execute(() =>
            {
                Console.WriteLine("Job Start");

                if (DateTime.Now.Second % 3 == 0)
                    throw new ArgumentException("Hello Polly!");
            });
        }
        catch (Exception ex)
        {
            // 手动打开熔断器，阻止执行
            breaker.Isolate();
        }
        Thread.Sleep(1000);

        // 恢复操作，启动执行
        breaker.Reset();
    }
}
```

## 降级 Fallback

```csharp
static void FallbackTest()
{
    ISyncPolicy policy = Policy.Handle<ArgumentException>()
    .Fallback(() =>
    {
        Console.WriteLine("Error occured,runing fallback");
    });

    policy.Execute(() =>
    {
        Console.WriteLine("Job Start");

        throw new ArgumentException("Hello Polly!");
    });
}
```

# 弹性应变处理 (Resilience)

## 超时

Timeout则是指超时处理，但是超时策略一般不能直接使用，而是其其他策略封装到一起使用。
常用悲观超时。

### 乐观超时 (Optimistic timeout)

```csharp
public void Should_be_able_to_cancel_with_user_cancellation_token_before_timeout__optimistic()
{
    int timeout = 10;
    var policy = Policy.TimeoutAsync(timeout, TimeoutStrategy.Optimistic);

    using (CancellationTokenSource userTokenSource = new CancellationTokenSource())
    {
        policy.Awaiting(async p => await p.ExecuteAsync(
            ct => {
                userTokenSource.Cancel(); ct.ThrowIfCancellationRequested();   // Simulate cancel in the middle of execution
                return TaskHelper.EmptyTask;
            }, userTokenSource.Token) // ... with user token.
           ).ShouldThrow<OperationCanceledException>();
    }
}
```

### 悲观超时

```csharp
static void PessimisticTimeOutTest()
{
    CancellationTokenSource userCancellationSource = new CancellationTokenSource();

    ISyncPolicy fallback = Policy.Handle<Polly.Timeout.TimeoutRejectedException>()
         .Or<ArgumentException>()
         .Fallback(() =>
         {
             Console.WriteLine("Error occured,runing fallback");
         },
         (ex) => { Console.WriteLine($"Fallback exception:{ex.GetType().ToString()},Message:{ex.Message}"); });

    ISyncPolicy policyTimeout = Policy.Timeout(3, Polly.Timeout.TimeoutStrategy.Pessimistic);

    while (true)
    {
        var policyWrap = Policy.Wrap(fallback, policyTimeout);

        // (wraps the policies around any executed delegate: fallback outermost ... bulkhead innermost)
        policyWrap.Execute(() =>
        {
            Console.WriteLine("Job Start");

            if (DateTime.Now.Second % 2 == 0)
            {
                Thread.Sleep(3000);
                throw new ArgumentException("Hello Polly!");
            }
        });
        Thread.Sleep(300);
    }
}
```

## 舱壁 Bulkhead

参考：

[https://github.com/App-vNext/Polly](https://github.com/App-vNext/Polly)