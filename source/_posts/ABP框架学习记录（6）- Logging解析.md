---
title: ABP框架学习记录（6）- Logging解析
date: 2019-05-07 09:15:27
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（6）- Logging解析

ABP使用Castle日志记录工具，并且可以使用不同的日志类库，比如：Log4Net, NLog, Serilog... 等等。对于所有的日志类库，Castle提供了一个通用的接口来实现，我们可以很方便的处理各种特殊的日志库，而且当业务需要的时候，很容易替换日志组件。

`LogSeverity`：表示日志的严重性，枚举类型，定义了5个日志级别：Info，Debug，Warn，Error， Fatal。

`IHasLogSeverity`：封装了LogSeverity。`AbpAuthorizationException`，`AbpValidationException`，`UserFriendlyException` 实现了这个接口。`Loghelper` 在对 `Exception` 做log的时候可以方便的通过实现了`IHasLogSeverity`的 `Exception` 的实例获取到 `LogSeverity`，以确定log 级别。

`AbpAuthorizationException`:

![QQ截图20190507105847.png](/img/QQ截图20190507105847.png)

`LogHelper` 的实现：把继承 `IHasLogSeverity` 接口的对象，转换成 `IHasLogSeverity` 对象，并获取log级别。

![QQ截图20190507110228.png](/img/QQ截图20190507110228.png)

`LoggerExtensions`： 扩展了Castle的Ilogger接口的方法，封装更便捷的方法供Loghelper调用。

![QQ截图20190507110501.png](/img/QQ截图20190507110501.png)

web项目的application_start方法中注入logger实例：

![QQ截图20190507110701.png](/img/QQ截图20190507110701.png)

参考：

[ABP源码分析八：Logger集成](http://www.cnblogs.com/1zhk/p/5294466.html)