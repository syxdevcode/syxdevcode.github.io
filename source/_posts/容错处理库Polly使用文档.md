---
title: 容错处理库Polly使用文档
date: 2018-07-25 09:23:20
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

* 错误处理(fault handling)：重试、熔断、回退(降级)
* 弹性应变(resilience)：超时、舱壁、缓存

可以说错误处理是当错误已经发生时，防止由于该错误对整个系统造成更坏的影响而设置。而弹性应变，则在是错误发生前，
针对有可能发生错误的地方进行预先处理，从而达到保护整个系统的目地。

# Polly 错误处理使用三步曲

* 定义条件： 定义你要处理的 错误异常/返回结果
* 定义处理方式 ： 重试，熔断，回退
* 执行

```csharp
var policy = Policy
              .Handle<SomeExceptionType>()   // 定义条件
              .Retry(); // 定义处理方式

// 执行
policy.Execute(() => DoSomething());
```

# 错误处理(fault handling)

## 定义条件

### 定义异常策略

针对两种情况：错误异常和返回结果

```csharp
// Single exception type
Policy
  .Handle<HttpRequestException>()

// Single exception type with condition
Policy
  .Handle<SqlException>(ex => ex.Number == 1205)

// Multiple exception types
Policy
  .Handle<HttpRequestException>()
  .Or<OperationCanceledException>()

// Multiple exception types with condition
Policy
  .Handle<SqlException>(ex => ex.Number == 1205)
  .Or<ArgumentException>(ex => ex.ParamName == "example")

// Inner exceptions of ordinary exceptions or AggregateException, with or without conditions
Policy
  .HandleInner<HttpRequestException>()
  .OrInner<OperationCanceledException>(ex => ex.CancellationToken != myToken)
```

### 定义返回结果策略

```csharp
// Handle return value with condition 
Policy
  .HandleResult<HttpResponseMessage>(r => r.StatusCode == HttpStatusCode.NotFound)

// Handle multiple return values 
Policy
  .HandleResult<HttpResponseMessage>(r => r.StatusCode == HttpStatusCode.InternalServerError)
  .OrResult<HttpResponseMessage>(r => r.StatusCode == HttpStatusCode.BadGateway)

// Handle primitive return values (implied use of .Equals())
Policy
  .HandleResult<HttpStatusCode>(HttpStatusCode.InternalServerError)
  .OrResult<HttpStatusCode>(HttpStatusCode.BadGateway)

// Handle both exceptions and return values in one policy
HttpStatusCode[] httpStatusCodesWorthRetrying = {
   HttpStatusCode.RequestTimeout, // 408
   HttpStatusCode.InternalServerError, // 500
   HttpStatusCode.BadGateway, // 502
   HttpStatusCode.ServiceUnavailable, // 503
   HttpStatusCode.GatewayTimeout // 504
};
HttpResponseMessage result = Policy
  .Handle<HttpRequestException>()
  .OrResult<HttpResponseMessage>(r => httpStatusCodesWorthRetrying.Contains(r.StatusCode))
  .RetryAsync(...)
  .ExecuteAsync( /* some Func<Task<HttpResponseMessage>> */ )
```

## 定义处理方式

### 重试 Retry

当发生某种错误或者返回某种结果的时候进行重试。
Polly里面提供了以下几种重试机制

* 按次数重试
* 不断重试（直到成功）
* 等待之后按次数重试
* 等待之后不断重试（直到成功）

#### 按次数重试

```csharp
// Retry once
Policy
  .Handle<SomeExceptionType>()
  .Retry()

// Retry multiple times
Policy
  .Handle<SomeExceptionType>()
  .Retry(3)

// Retry multiple times, calling an action on each retry
// with the current exception and retry count
Policy
    .Handle<SomeExceptionType>()
    .Retry(3, (exception, retryCount) =>
    {
        // do something
    });

// Retry multiple times, calling an action on each retry
// with the current exception, retry count and context
// provided to Execute()
Policy
    .Handle<SomeExceptionType>()
    .Retry(3, (exception, retryCount, context) =>
    {
        // do something
    });
```

#### 不断重试(直到成功)

```csharp
// Retry forever
Policy
  .Handle<SomeExceptionType>()
  .RetryForever()

// Retry forever, calling an action on each retry with the
// current exception
Policy
  .Handle<SomeExceptionType>()
  .RetryForever(exception =>
  {
        // do something
  });

// Retry forever, calling an action on each retry with the
// current exception and context provided to Execute()
Policy
  .Handle<SomeExceptionType>()
  .RetryForever((exception, context) =>
  {
        // do something
  });
```

#### 等待之后重试

```csharp
// Retry, waiting a specified duration between each retry
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(new[]
  {
    TimeSpan.FromSeconds(1),
    TimeSpan.FromSeconds(2),
    TimeSpan.FromSeconds(3)
  });

// Retry, waiting a specified duration between each retry,
// calling an action on each retry with the current exception
// and duration
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(new[]
  {
    TimeSpan.FromSeconds(1),
    TimeSpan.FromSeconds(2),
    TimeSpan.FromSeconds(3)
  }, (exception, timeSpan) => {
    // do something
  });

// Retry, waiting a specified duration between each retry,
// calling an action on each retry with the current exception,
// duration and context provided to Execute()
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(new[]
  {
    TimeSpan.FromSeconds(1),
    TimeSpan.FromSeconds(2),
    TimeSpan.FromSeconds(3)
  }, (exception, timeSpan, context) => {
    // do something
  });

// Retry, waiting a specified duration between each retry,
// calling an action on each retry with the current exception,
// duration, retry count, and context provided to Execute()
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(new[]
  {
    TimeSpan.FromSeconds(1),
    TimeSpan.FromSeconds(2),
    TimeSpan.FromSeconds(3)
  }, (exception, timeSpan, retryCount, context) => {
    // do something
  });

// Retry a specified number of times, using a function to 
// calculate the duration to wait between retries based on 
// the current retry attempt (allows for exponential backoff)
// In this case will wait for
//  2 ^ 1 = 2 seconds then
//  2 ^ 2 = 4 seconds then
//  2 ^ 3 = 8 seconds then
//  2 ^ 4 = 16 seconds then
//  2 ^ 5 = 32 seconds
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(5, retryAttempt =>
    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))
  );

// Retry a specified number of times, using a function to
// calculate the duration to wait between retries based on
// the current retry attempt, calling an action on each retry
// with the current exception, duration and context provided
// to Execute()
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(
    5,
    retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
    (exception, timeSpan, context) => {
      // do something
    }
  );

// Retry a specified number of times, using a function to
// calculate the duration to wait between retries based on
// the current retry attempt, calling an action on each retry
// with the current exception, duration, retry count, and context
// provided to Execute()
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetry(
    5,
    retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
    (exception, timeSpan, retryCount, context) => {
      // do something
    }
  );
```

#### 等待并永远重试（直到成功）

```csharp
// Wait and retry forever
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetryForever(retryAttempt =>
    TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))
    );

// Wait and retry forever, calling an action on each retry with the
// current exception and the time to wait
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetryForever(
    retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
    (exception, timespan) =>
    {
        // do something
    });

// Wait and retry forever, calling an action on each retry with the
// current exception, time to wait, and context provided to Execute()
Policy
  .Handle<SomeExceptionType>()
  .WaitAndRetryForever(
    retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
    (exception, timespan, context) =>
    {
        // do something
    });
```

如果所有重试都失败，则重试策略会将最终异常重新返回到调用代码。

### 熔断 Circuit Breaker

````csharp
// 当发生2次SomeExceptionType的异常的时候则会熔断1分钟，
// 该操作后续如果继续尝试执行则会直接返回错误 。
Policy
    .Handle<SomeExceptionType>()
    .CircuitBreaker(2, TimeSpan.FromMinutes(1));

// 可以在熔断和恢复的时候定义委托来做一些额外的处理。
// onBreak会在被熔断时执行，而onReset则会在恢复时执行。
Action<Exception, TimeSpan> onBreak = (exception, timespan) => { ... };
Action onReset = () => { ... };
CircuitBreakerPolicy breaker = Policy
    .Handle<SomeExceptionType>()
    .CircuitBreaker(2, TimeSpan.FromMinutes(1), onBreak, onReset);

// Break the circuit after the specified number of consecutive exceptions
// and keep circuit broken for the specified duration,
// calling an action on change of circuit state,
// passing a context provided to Execute().
Action<Exception, TimeSpan, Context> onBreak = (exception, timespan, context) => { ... };
Action<Context> onReset = context => { ... };
CircuitBreakerPolicy breaker = Policy
    .Handle<SomeExceptionType>()
    .CircuitBreaker(2, TimeSpan.FromMinutes(1), onBreak, onReset);

// Monitor the circuit state, for example for health reporting.
CircuitState state = breaker.CircuitState;

/*
Closed 关闭状态，允许执行
Open 自动打开，执行会被阻断
Isolate 手动打开，执行会被阻断
HalfOpen  从自动打开状态恢复中，在熔断时间到了之后从Open状态切换到Closed
*/

// 手动打开熔断器，阻止执行
breaker.Isolate();

// 恢复操作，启动执行
breaker.Reset();
````

请注意，断路器策略会重新抛出所有异常，甚至是已处理的异常。断路器用于测量故障并在发生太多故障时断开电路，
但不会重新编排重试。根据需要将断路器与重试策略相结合。

### 回退（Fallback)

````csharp
// 如果执行失败则返回UserAvatar.Blank
Policy
   .Handle<Whatever>()
   .Fallback<UserAvatar>(UserAvatar.Blank)

// 发起另外一个请求去获取值
Policy
   .Handle<Whatever>()
   .Fallback<UserAvatar>(() => UserAvatar.GetRandomAvatar())
   // where: public UserAvatar GetRandomAvatar() { ... }

// 返回一个指定的值，添加额外的处理操作。onFallback
Policy
   .Handle<Whatever>()
   .Fallback<UserAvatar>(UserAvatar.Blank, onFallback: (exception, context) =>
    {
        // do something
    });
````

## 执行(Execute the policy)

````csharp
// Execute an action
var policy = Policy
              .Handle<SomeExceptionType>()
              .Retry();

policy.Execute(() => DoSomething());

// Execute an action passing arbitrary context data
var policy = Policy
    .Handle<SomeExceptionType>()
    .Retry(3, (exception, retryCount, context) =>
    {
        var methodThatRaisedException = context["methodName"];
        Log(exception, methodThatRaisedException);
    });

policy.Execute(
    (o) => DoSomething(),
    new Dictionary<string, object>() {{ "methodName", "some method" }}
);

// Execute a function returning a result
var policy = Policy
              .Handle<SomeExceptionType>()
              .Retry();

var result = policy.Execute(() => DoSomething());

// Execute a function returning a result passing arbitrary context data
var policy = Policy
    .Handle<SomeExceptionType>()
    .Retry(3, (exception, retryCount, context) =>
    {
        object methodThatRaisedException = context["methodName"];
        Log(exception, methodThatRaisedException)
    });

var result = policy.Execute(
    () => DoSomething(),
    new Dictionary<string, object>() {{ "methodName", "some method" }}
);

// 我们也可以将Handle，Retry, Execute 这三个阶段都串起来写。
Policy
  .Handle<SqlException>(ex => ex.Number == 1205)
  .Or<ArgumentException>(ex => ex.ParamName == "example")
  .Retry()
  .Execute(() => DoSomething());
````

# Polly 弹性应变处理 (Resilience)

一般弹性策略添加了弹性策略，这些策略没有明确地以处理委托可能抛出或返回的故障为中心。

## 超时

### 乐观超时 (Optimistic timeout)

超时分为乐观超时与悲观超时，乐观超时依赖于CancellationToken ，它假设我们的具体执行的任务都支持CancellationToken。
那么在进行timeout的时候，它会通知执行线程取消并终止执行线程，避免额外的开销。下面的乐观超时的具体用法 。

```csharp
// Timeout and return to the caller after 30 seconds, if the executed delegate has not completed.  
// Optimistic timeout: Delegates should take and honour a CancellationToken.
Policy
  .Timeout(30)

// Configure timeout as timespan.
Policy
  .Timeout(TimeSpan.FromMilliseconds(2500))

// Configure variable timeout via a func provider.
Policy
  .Timeout(() => myTimeoutProvider)) // Func<TimeSpan> myTimeoutProvider

// Timeout, calling an action if the action times out
Policy
  .Timeout(30, onTimeout: (context, timespan, task) =>
    {
        // do something
    });

// Eg timeout, logging that the execution timed out:
Policy
  .Timeout(30, onTimeout: (context, timespan, task) =>
    {
        logger.Warn($"{context.PolicyKey} at {context.ExecutionKey}: execution timed out after {timespan.TotalSeconds} seconds.");
    });

// Eg timeout, capturing any exception from the timed-out task when it completes:
Policy
  .Timeout(30, onTimeout: (context, timespan, task) =>
    {
        task.ContinueWith(t => {
            if (t.IsFaulted)
               logger.Error($"{context.PolicyKey} at {context.ExecutionKey}: execution timed out after {timespan.TotalSeconds} seconds, with: {t.Exception}.");
        });
    });

Policy timeoutPolicy = Policy.TimeoutAsync(30);
HttpResponseMessage httpResponse = await timeoutPolicy
    .ExecuteAsync(
      async ct => await httpClient.GetAsync(endpoint, ct), // Execute a delegate which responds to a CancellationToken input parameter.
      CancellationToken.None
       // In this case, CancellationToken.None is passed into the execution, indicating you have no independent cancellation
       // control you wish to add to the cancellation provided by TimeoutPolicy.  Your own indepdent CancellationToken can also
       // be passed - see wiki for examples.
      );


CancellationTokenSource userCancellationSource = new CancellationTokenSource();
// userCancellationSource perhaps hooked up to the user clicking a 'cancel' button, or other independent cancellation

Policy timeoutPolicy = Policy.TimeoutAsync(30, TimeoutStrategy.Optimistic);

HttpResponseMessage httpResponse = await _timeoutPolicy
    .ExecuteAsync(
        async ct => await httpClient.GetAsync(requestEndpoint, ct),
        userCancellationSource.Token
        ); // GetAsync(...) will be cancelled when either the timeout occurs, or userCancellationSource is signalled.
```

### 悲观超时

悲观超时与乐观超时的区别在于，如果执行的代码不支持取消CancellationToken，它还会继续执行，这会是一个比较大的开销。

```csharp
Policy timeoutPolicy = Policy.TimeoutAsync(30, TimeoutStrategy.Pessimistic);
var response = await timeoutPolicy
    .ExecuteAsync(
      async () => await FooNotHonoringCancellationAsync(),
      );// 在这里我们没有 任何与CancllationToken相关的处理
```

## 舱壁 Bulkhead

```csharp
// 它用来限制某一个操作的最大并发执行数量 。比如限制为12
Policy
  .Bulkhead(12)

// 控制一个等待处理的队列长度
Policy
  .Bulkhead(12, 2)

// 当请求执行操作被拒绝的时候，执行回调
Policy
  .Bulkhead(12, context =>
    {
        // do something
    });

// Monitor the bulkhead available capacity, for example for health/load reporting.
var bulkhead = Policy.Bulkhead(12, 2);
// ...
int freeExecutionSlots = bulkhead.BulkheadAvailableCount;
int freeQueueSlots     = bulkhead.QueueAvailableCount;
```

## 缓存 Cache

```csharp
Microsoft.Extensions.Caching.Memory.IMemoryCache memoryCache
   = new Microsoft.Extensions.Caching.Memory.MemoryCache(new Microsoft.Extensions.Caching.Memory.MemoryCacheOptions());
Polly.Caching.Memory.MemoryCacheProvider memoryCacheProvider
   = new Polly.Caching.Memory.MemoryCacheProvider(memoryCache);

// (2) Create a Polly cache policy using that Polly.Caching.Memory.MemoryCacheProvider instance.
var cachePolicy = Policy.Cache(memoryCacheProvider, TimeSpan.FromMinutes(5));
```

参考代码：

[https://github.com/App-vNext/Polly.Caching.MemoryCache](https://github.com/App-vNext/Polly.Caching.MemoryCache)
[https://github.com/App-vNext/Polly.Caching.IDistributedCache](https://github.com/App-vNext/Polly.Caching.IDistributedCache)

## 组合Policy

```csharp
// Define a combined policy strategy, built of previously-defined policies.
var policyWrap = Policy
  .Wrap(fallback, cache, retry, breaker, timeout, bulkhead);
// (wraps the policies around any executed delegate: fallback outermost ... bulkhead innermost)
policyWrap.Execute(...)

// Define a standard resilience strategy ...
PolicyWrap commonResilience = Policy.Wrap(retry, breaker, timeout);

// ... then wrap in extra policies specific to a call site, at that call site:
Avatar avatar = Policy
   .Handle<Whatever>()
   .Fallback<Avatar>(Avatar.Blank)
   .Wrap(commonResilience)
   .Execute(() => { /* get avatar */ });

// Share commonResilience, but wrap different policies in at another call site:
Reputation reps = Policy
   .Handle<Whatever>()
   .Fallback<Reputation>(Reputation.NotAvailable)
   .Wrap(commonResilience)
   .Execute(() => { /* get reputation */ });  
```

参考代码：[https://github.com/App-vNext/Polly/wiki/PolicyWrap](https://github.com/App-vNext/Polly/wiki/PolicyWrap)

# 执行后：捕获结果或任何最终异常

```csharp
var policyResult = Policy
              .Handle<HttpRequestException>()
              .RetryAsync()
              .ExecuteAndCaptureAsync(() => DoSomethingAsync());
/*
policyResult.Outcome - whether the call succeeded or failed
policyResult.FinalException - the final exception captured, will be null if the call succeeded
policyResult.ExceptionType - was the final exception an exception the policy was defined to handle
 (like HttpRequestException above) or an unhandled one (say Exception). Will be null if the call succeeded.
policyResult.Result - if executing a func, the result if the call succeeded or the type's default value
*/
```

参考：

[https://github.com/App-vNext/Polly](https://github.com/App-vNext/Polly)

[ASP VNext 开源服务容错处理库Polly使用文档](https://www.cnblogs.com/jesse2013/p/polly-docs.html)

[已被.NET基金会认可的弹性和瞬态故障处理库Polly介绍](http://www.cnblogs.com/CreateMyself/p/7589397.html)

[.NET Core微服务之基于Polly+AspectCore实现熔断与降级机制](https://www.cnblogs.com/edisonchou/p/9159644.html)