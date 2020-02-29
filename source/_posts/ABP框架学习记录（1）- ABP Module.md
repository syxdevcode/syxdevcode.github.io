---
title: ABP框架学习记录（1）- ABP Module
date: 2019-04-29 14:11:48
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---

Abp是一种基于模块化设计的思想构建的。开发人员可以将自定义的功能以模块（module）的形式集成到ABP中。具体的功能都可以设计成一个单独的Module。Abp底层框架提供便捷的方法集成每个Module。

## AbpModule

在 `Abp` 项目下的 `Abp.Modules` 文件夹下定义了抽象类 `AbpModule` ，它提供了两个受保护的属性和四个虚方法：

```cs
/// <summary>
/// IOC 管理
/// </summary>
protected internal IIocManager IocManager { get; internal set; }

/// <summary>
/// 配置
/// </summary>
protected internal IAbpStartupConfiguration Configuration { get; internal set; }

/// <summary>
/// 应用程序启动时调用的第一个事件，在依赖注入之前运行
/// </summary>
public virtual void PreInitialize(){}

/// <summary>
/// 注册依赖项
/// </summary>
public virtual void Initialize(){}

/// <summary>
/// 应用程序启动时调用
/// </summary>
public virtual void PostInitialize(){}

/// <summary>
/// 关闭应用程序时调用
/// </summary>
public virtual void Shutdown(){}
```

## DependsOnAttribute

`AbpModule` 还提供 `FindDependedModuleTypes` 方法，获取使用 `DependsOnAttribute` 属性的
`Module` 集合：

![QQ截图20190429152244.png](/img/QQ截图20190429152244.png)

## AbpModuleManager

`Abp.Modules` 文件夹下定义了 `AbpModuleManager` 类，来管理 `ABPModule`：

![QQ截图20190429161029.png](/img/QQ截图20190429161029.png)

## AbpModuleCollection

`AbpModuleCollection` 类继承 `List<AbpModuleInfo>`：

![QQ截图20190429162910.png](/img/QQ截图20190429162910.png)

`AbpModuleManager` 得到所有的 `AbpModule` 的 `AbpModuleInfo` 以后，逐个调用这些 `Module` 的   `PreInitialize` ，`Initialize` 和 `PostInitialize` 以完成初始化：

![QQ截图20190430132848.png](/img/QQ截图20190430132848.png)

```cs
public virtual void StartModules()
{
    var sortedModules = _modules.GetSortedModuleListByDependency();
    sortedModules.ForEach(module => module.Instance.PreInitialize());
    sortedModules.ForEach(module => module.Instance.Initialize());
    sortedModules.ForEach(module => module.Instance.PostInitialize());
}
```

## AbpKernelModule

Abp底层框架的一些功能模块的类型通过 `AbpKernelModule` 实现:

![QQ截图20190429163427.png](/img/QQ截图20190429163427.png)

![QQ截图20190429163852.png](/img/QQ截图20190429163852.png)

参考：

[ABP源码分析三：ABP Module ](http://www.cnblogs.com/1zhk/p/5281458.html)