---
title: ABP框架学习记录（4）- Configuration解析
date: 2019-05-05 17:26:31
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（4）- Configuration解析

目录结构：

![QQ截图20190506094754.png](/img/QQ截图20190506094754.png)

通过 `AbpStartupConfiguration`，Castle的依赖注入，`Dictionary` 对象和扩展方法实现了配置中心化。配置中心化是一个支持模块开发的框架必备功能。

## 核心模块配置

### 模块配置

ABP中核心功能模块中的一些功能的运行时的行为依赖于一些外部配置。比如 `Localization` 这个功能模块，最基本Abp需要知道要做哪些语言的本地化。而这些具体的配置对于Abp底层框架来说是不可预知的，那么ABP底层框架就很有必要提供一种手段供外部模块自定义 `Congfiguration`。 这就是下文要分析的 `IAbpStartupConfiguration` 和各种`I***Configuration`。

通过 `AbpBootstrapper` 的 `Initialize` 方法，注册 `AbpCoreInstaller`:

具体请参考：[ABP框架学习记录（2） - ABP初始化](https://syxdevcode.github.io/2019/04/30/ABP框架学习记录（2）-%20ABP初始化/)

![QQ截图20190506101758.png](/img/QQ截图20190506101758.png)

`AbpCoreInstaller`:

![QQ截图20190506101856.png](/img/QQ截图20190506101856.png)

`IAbpStartupConfiguration`/`AbpStartupConfiguration` :  `Initialize` 方法，通过调用容器，获取基础配置实例。

![QQ截图20190506102036.png](/img/QQ截图20190506102036.png)

### 调用配置

抽象类 `AbpModule` 提供 `Configuration` 字段，继承 `AbpModule` 的子类，可以直接调用或修改某个组件的`Configuration` 。

`AbpModule`：

![QQ截图20190506102516.png](/img/QQ截图20190506102516.png)

调用/修改：

![QQ截图20190506102855.png](/img/QQ截图20190506102855.png)

## 自定义模块配置

### 注册

以 `IAbpWebCommonModuleConfiguration` 为例：

定义 `IAbpWebCommonModuleConfiguration` 接口：

![QQ截图20190506113226.png](/img/QQ截图20190506113226.png)

`AbpWebCommonModule` 中的 `PreInitialize` 方法注册实例：

![QQ截图20190506113321.png](/img/QQ截图20190506113321.png)

### 获取

`DictionaryBasedConfig` 定义的 `CustomSettings` 就是最终保存自定义的module的Configuration的地方。

![QQ截图20190506111029.png](/img/QQ截图20190506111029.png)

`AbpStartupConfiguration` 继承 `DictionaryBasedConfig`：

![QQ截图20190506111449.png](/img/QQ截图20190506111449.png)

`IModuleConfigurations` 中定义 `IAbpStartupConfiguration`：

![QQ截图20190506111717.png](/img/QQ截图20190506111717.png)

`ModuleConfigurations` 的实现：

![QQ截图20190506113929.png](/img/QQ截图20190506113929.png)

`AbpWebConfigurationExtensions` 提供对 `IModuleConfigurations` 的扩展：

![QQ截图20190506112702.png](/img/QQ截图20190506112702.png)

`AbpStartupConfiguration` 获取配置方法：

![QQ截图20190506131005.png](/img/QQ截图20190506131005.png)

`DictionaryBasedConfig`中定义的方法：

![QQ截图20190506131238.png](/img/QQ截图20190506131238.png)

使用 `AbpWebConfigurationExtensions`：

![QQ截图20190506113848.png](/img/QQ截图20190506113848.png)

参考：

[ABP源码分析四：Configuration](http://www.cnblogs.com/1zhk/p/5285623.html)