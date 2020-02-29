---
title: ABP框架学习记录（5）- Setting解析
date: 2019-05-06 13:30:09
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（5）- Setting解析

ABP中的 `Setting` 和 `Configuration`：

Setting一般用于需要通过外部配置文件（或数据库）设置的简单类型数据（一般就是字符串）。

Configuration一般只需要通过内部代码完成的配置，一般用于设置复杂类型的数据。

## Setting的实现

`SettingDefinition`：用于定义Setting。

`SettingDefinitionGroup`：用于给`SettingDefinition`分组。

`SettingsConfiguration` / `ISettingsConfiguration`：用于集中化设置和管理 `SettingProvider` 的对象。其封装了一个`ITypeList<SettingProvider> Providers`的集合类。可以通过`Configuration.Setting`来获取`ISettingsConfiguration`实例，然后将自定义的 `SettingProvider` 添加到 `SettingsConfiguration` 对象中，下图：

![QQ截图20190506162020.png](/img/QQ截图20190506162020.png)

`SettingDefinitionManager`：主要完成注册到ABP中的 `SettingDefinition` 初始化，通过 `ISettingsConfiguration` 实例获取 `setting providers` 集合，然后在 `Initialize` 方法中通过 `setting providers` 获取 `SettingDefinition` 的数组。并将其保存在 `Dictionary` 中，其key就是 `SettingDefinition` 的Name，下图：

![QQ截图20190506162528.png](/img/QQ截图20190506162528.png)

`SettingDefinitionManager` 继承 `ISingletonDependency`接口，将在`BasicConventionalRegistrar`类中实现约定规则的注入：

![QQ截图20190506163107.png](/img/QQ截图20190506163107.png)

`AbpKernelModule` 中 `PostInitialize` 调用 `SettingDefinitionManager`的 `Initialize`方法：

![QQ截图20190506173110.png](/img/QQ截图20190506173110.png)

`SettingDefinitionProviderContext`：用于对 `ISettingDefinitionManager` 的封装：

![QQ截图20190506162348.png](/img/QQ截图20190506162348.png)

`SettingScopes`：标注了Flags特性的枚举类型，表示setting的应用范围：

![QQ截图20190506162316.png](/img/QQ截图20190506162316.png)

`SettingProvider`：为具体的功能模块所需的设置定义 `SettingDefinition`，并且以数组的形式返回：

![QQ截图20190506170342.png](/img/QQ截图20190506170342.png)

`SettingManager` / `ISettingManager` ：用户获取配置详情。

![QQ截图20190506172935.png](/img/QQ截图20190506172935.png)

公共方法：

![QQ截图20190506173541.png](/img/QQ截图20190506173541.png)

## 以mail的实现来使用Setting

目录结构：

![QQ截图20190506173847.png](/img/QQ截图20190506173847.png)

`IEmailSenderConfiguration`/`EmailSenderConfiguration` 和 `ISmtpEmailSenderConfiguration`/`SmtpEmailSenderConfiguration`：定义配置。

`EmailSenderConfiguration`：

![QQ截图20190509145449.png](/img/QQ截图20190509145449.png)

`SmtpEmailSenderConfiguration`：

![QQ截图20190509145529.png](/img/QQ截图20190509145529.png)

`EmailSettingProvider`：继承自 `SettingProvider`, 将SMTP的各项设置封装成`SettingDefinition`，并以数组形式返回。

![QQ截图20190509145337.png](/img/QQ截图20190509145337.png)

`EmailSenderBase`/`IEmailSender` 和 `ISmtpEmailSender`/`SmtpEmailSender`：用户发送邮件。

参考：

[ABP源码分析七：Setting 以及 Mail](http://www.cnblogs.com/1zhk/p/5299469.html)