---
title: IHostedService后台任务摘要
date: 2020-01-15 11:14:02
tags:
- CSharp
- DotNetCore
- HostService
categories: 
- HostService
---
## 后台任务

摘自：[Background tasks with hosted services in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-3.1&tabs=visual-studio)

BackgroundService is a base class for implementing a long running IHostedService.

ExecuteAsync(CancellationToken) is called to run the background service. The implementation returns a Task that represents the entire lifetime of the background service. No further services are started until ExecuteAsync becomes asynchronous, such as by calling . Avoid performing long, blocking initialization work in . The host blocks in StopAsync(CancellationToken) waiting for to complete.awaitExecuteAsyncExecuteAsync

The cancellation token is triggered when IHostedService.StopAsync is called. Your implementation of should finish promptly when the cancellation token is fired in order to gracefully shut down the service. Otherwise, the service ungracefully shuts down at the shutdown timeout. For more information, see the IHostedService interface section.ExecuteAsync

## 总结

当继承 `IHostedService` 接口实现 `BackgroundService` 或者 `BackgroundTasks` ，其实现类在注入时，不受注入的先后顺序影响。

下面的例子，`ConsumerManager` 依赖 `IKafkaEventBusContainer` 的实现：

```cs
serviceCollection.AddHostedService<ConsumerManager>();
serviceCollection.AddSingleton<IKafkaEventBusContainer, EventBusContainer>();
```

参考：

[Background tasks with hosted services in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-3.1&tabs=visual-studio)
