---
title: ABP框架学习记录（2）- ABP初始化
date: 2019-04-30 09:18:00
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（2）- ABP初始化

ASP.NET Web应用程序的第一个执行的方法是 Global.asax 下定义的Start方法。执行这个方法前 `HttpApplication` 实例必须存在，也就是说其构造函数的执行必然是完成了。

## AbpWebApplication

Global.asax 中 `MvcApplication` 继承自泛型 `AbpWebApplication<>` ,并提供 `AbpZeroTemplateWebModule` 作为 `StartupModule`。

![QQ截图20190430093710.png](/img/QQ截图20190430093710.png)

泛型 `AbpWebApplication<>` (下图)，继承自 `HttpApplication`，实例化 `AbpBootstrapper` 对象:

![QQ截图20190430134242.png](/img/QQ截图20190430134242.png)

## AbpBootstrapper

在 `AbpBootstrapper` 的构造函数中，实例化 `AbpBootstrapperOptions` 对象，以提供 `IocManager` 实例：

![QQ截图20190430135239.png](/img/QQ截图20190430135239.png)

`AbpBootstrapperOptions` 类：

![QQ截图20190430135457.png](/img/QQ截图20190430135457.png)

`AbpBootstrapper` 类提供私有构造函数，并且提供泛型 `Create` 方法以创建实例：

![QQ截图20190510165701.png](/img/QQ截图20190510165701.png)

`Create` ：方法

![QQ截图20190510165854.png](/img/QQ截图20190510165854.png)

拦截器注册：

![QQ截图20190510170025.png](/img/QQ截图20190510170025.png)

![QQ截图20190510170146.png](/img/QQ截图20190510170146.png)

![QQ截图20190510170458.png](/img/QQ截图20190510170458.png)

1，`Initialize` 方法：

![QQ截图20190430140353.png](/img/QQ截图20190430140353.png)

2，`Initialize` 作用：

(1)，安装 `AbpCoreInstaller`； `AbpCoreInstaller` 的作用是用来注册系统框架级的所有配置类。

`AbpCoreInstaller` 类：

![QQ截图20190430140706.png](/img/QQ截图20190430140706.png)

(2)，增加插件

```cs
IocManager.Resolve<AbpPlugInManager>().PlugInSources.AddRange(PlugInSources);
```

(3)，初始化配置

![QQ截图20190430143142.png](/img/QQ截图20190430143142.png)

(4)，通过 `AbpModuleManager` 管理 `AdpModule`;

```cs
_moduleManager = IocManager.Resolve<AbpModuleManager>();
_moduleManager.Initialize(StartupModule);
_moduleManager.StartModules();
```

参考：

[ABP源码分析二：ABP中配置的注册和初始化 ](http://www.cnblogs.com/1zhk/p/5277610.html)