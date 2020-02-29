---
title: ABP框架学习记录（7）- 后台工作任务
date: 2019-05-08 09:23:29
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（7）- 后台工作任务

文主要说明ABP中后台工作者模块（BackgroundWorker）的实现方式，和后台工作模块（BackgroundJob）。
ABP通过 `BackgroundWorkerManager` 来管理 `BackgroundJobManager` ，然后通过 `BackgroundJobManager` 来管理 `BackgroundJob`。`BackgroundJob` 就代表一个真正的后台任务。

`AbpKernelModule` 的 `PostInitialize` 方法完成对 `BackgroundWorkerManager`，`BackgroundJobManager`的初始化。

![QQ截图20190508151157.png](/img/QQ截图20190508151157.png)

## BackgroundWorkerManager 工作者

目录结构：

![QQ截图20190508151546.png](/img/QQ截图20190508151546.png)

`IRunnable/RunnableBase`：定义了启动/终止一个任务的方法的接口和基本实现抽象类。共三个方法：start, stop, waittostop。 

![QQ截图20190508151806.png](/img/QQ截图20190508151806.png)

`BackgroundWorkerBase`：实现 `IBackgroundWorker` 的一个抽象类，同时添加了UOW,Setting 和本地化的一些辅助方法。

![QQ截图20190508152210.png](/img/QQ截图20190508152210.png)

`PeriodicBackgroundWorkerBase`：继承 `BackgroundWorkerBase`， 通过封装 `AbpTimer` 实现定时启动执行任务的功能。这个类型定义个一个抽象方法 `DoWork`。`AbpTimer` 最终会定时执行这个方法。

![QQ截图20190508161027.png](/img/QQ截图20190508161027.png)

![QQ截图20190508162829.png](/img/QQ截图20190508162829.png)

<font color=#ff0000 size=4 face="黑体">AbpTimer是整个ABP框架实现后台工作的核心类，其实现原理就是通过一个CLR中的timer定时启动执行任务。</font>

使用Timer需要注意的问题：

第一，用timer有一个弊端，就是当timer间隔时间内，任务如果没执行完，timer就会新建一个线程，从头开始执行这个任务，而上一个线程仍然继续执行，这样就会导致系统中产生的线程过多，一会儿系统的资源就耗尽了。ABP的解决思路是在执行真正的业务方法之前，通过将timer的duetime设为无限大，从而timer就失效了。业务方法执行完以后在恢复timer的设置。

![QQ截图20190508173455.png](/img/QQ截图20190508173455.png)

`Timer` 官方文档解释为：

![QQ截图20190508173557.png](/img/QQ截图20190508173557.png)

[System.Threading.Timer](https://docs.microsoft.com/zh-cn/dotnet/api/system.threading.timer.change?view=netframework-4.8)

第二，如何知道一个Timer真正结束了呢？也就是说如何知道一个Timer要执行的任务已经完成（定义为A效果），同时timer已失效(定义为B效果)？ABP通过stop方法实现B，通过WaitToStop实现A效果。WaitToStop会一直阻塞调用他的线程直到_performingTasks变成false,也就是说Timer要执行的任务已经完成（任务完成时会将_performingTasks设为False，并且释放锁）。

![QQ截图20190508174549.png](/img/QQ截图20190508174549.png)

```cs
/// <summary>
/// _taskTimer的回调方法
/// </summary>
/// <param name="state">Not used argument</param>
private void TimerCallBack(object state)
{
    lock (_taskTimer)
    {
        if (!_running || _performingTasks)
        {
            return;
        }

        _taskTimer.Change(Timeout.Infinite, Timeout.Infinite);
        _performingTasks = true;
    }

    try
    {
        if (Elapsed != null)
        {
            Elapsed(this, new EventArgs());
        }
    }
    catch
    {}
    finally
    {
        lock (_taskTimer)
        {
            _performingTasks = false;
            if (_running)
            {
                // 指示Timer继续执行
                _taskTimer.Change(Period, Timeout.Infinite);
            }
            // 释放锁
            Monitor.Pulse(_taskTimer);
        }
    }
}
```

`IBackgroundWorkerManager/BackgroundWorkerManager`：用于管理后台工作任务：`IBackgroundWorker` 实例（添加 `IBackgroundWorker` 实例到管理器，启动，终止和注销后台任务）。

![QQ截图20190508160425.png](/img/QQ截图20190508160425.png)

## BackgroundWorkerManager 工作者

在ABP项目的目录结构：

![QQ截图20190509104729.png](/img/QQ截图20190509104729.png)

`IBackgroundJob/BackgroundJob`：定义一个后台工作任务的接口/和基本实现。具体的后台任务类可从 `BackgroundJob` 继承，这是定义最终需要被执行的逻辑的地方。

![QQ截图20190509113529.png](/img/QQ截图20190509113529.png)

`BackgroundJobInfo`：用于持久化job信息的实体类，对应于数据库中的表 `AbpBackgroundJobs`。一个job对应一个要执行的任务。

![QQ截图20190509131552.png](/img/QQ截图20190509131552.png)

`IBackgroundJobConfiguration/BackgroundJobConfiguration`：配置是否激活后台工作任务功能。

![QQ截图20190509114407.png](/img/QQ截图20190509114407.png)

`BackgroundJobPriority`：后台job的优先级。

`IBackgroundJobStore/InMemoryBackgroundJobStore`: 用于持久化后台任务 `BackgroundJobInfo`。可以实现这个接口将后台任务 `BackgroundJobInfo` 存储到数据库。

`IBackgroundJobManager/BackgroundJobManager`：`IBackgroundJobManager` 继承 `IBackgroundWorker`，`BackgroundJobManager` 默认实现 `IBackgroundJobManager` ，并 继承 `PeriodicBackgroundWorkerBase`。它可以被其他的后台工作提供者替代（Hangfire）。`BackgroundJobManager`之所以能在后台执行任务，是因为其继承了`PeriodicBackgroundWorkerBase` 基类，并重写了DoWork方法。从 `BackgroundJobStore`（可以自定义实现从数据库中读取）取最多1000个 `BackgroundJobInfo`，然后反射执行 `BackgroundJobInfo` 中定义的任务。

![QQ截图20190509131058.png](/img/QQ截图20190509131058.png)

![QQ截图20190509131329.png](/img/QQ截图20190509131329.png)

下面是一个ABP中通过 `BackgroundJobManager` 安排 `BackgroundJob` 的例子。

![QQ截图20190509132129.png](/img/QQ截图20190509132129.png)

参考：

[ABP源码分析九：后台工作任务](http://www.cnblogs.com/1zhk/p/5304244.html)