---
title: ABP框架学习记录（14）- Auditing的实现
date: 2019-07-24 09:23:27
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（14）- Auditing的实现

**Auditing**：<font color=#ff0000 size=4 face="黑体">审计跟踪（也叫审计日志）是与安全相关的信息按照时间顺序的记录，它们提供了活动序列的文档证据，这些活动序列可以在任何时间影响一个特定的操作。</font>

Auditing 在 ABP 项目中的位置：

![QQ截图20190727141502.png](/img/QQ截图20190727141502.png)

## 基础定义

`AuditInfo`：定义需要被审计信息实体类；

`AuditedAttribute`：用于标识一个方法或一个类的所有方法都需要启用Auditing功能。

`DisableAuditingAttribute`：用于标识一个方法或一个类的所有方法都需要关闭Auditing功能。

`IAuditSerializer`：提供审计序列化接口；

`JsonNetAuditSerializer`：`IAuditSerializer` 接口的默认实现，提供序列化方法；

`IAuditInfoProvider`：提供 `Fill` 方法，完善 `AuditInfo` 信息。

`DefaultAuditInfoProvider`：`IAuditInfoProvider` 的默认实现；

`IClientInfoProvider`：提供客户端信息的接口；

`NullClientInfoProvider`：`IClientInfoProvider` 接口的默认实现；

`IAuditingStore`：审计信息持久化接口，应由上层提供商实现；

`SimpleLogAuditingStore`：`IAuditingStore` 接口的默认实现；

## 入口

`AbpBootstrapper` 提供统一的拦截器注入入口：

![QQ截图20190727150235.png](/img/QQ截图20190727150235.png)

![QQ截图20190727150303.png](/img/QQ截图20190727150303.png)

`AuditingInterceptorRegistrar`：审计拦截器注册类，如下图：

![QQ截图20190727150506.png](/img/QQ截图20190727150506.png)

`AuditingInterceptor`：审计拦截器,提供对同步方法和异步方法的拦截操作，如下图：

![QQ截图20190727152548.png](/img/QQ截图20190727152548.png)

![QQ截图20190727153054.png](/img/QQ截图20190727153054.png)

![QQ截图20190727153419.png](/img/QQ截图20190727153419.png)

`IAuditingHelper`：记录审计信息帮助类；

`AuditingHelper`：`IAuditingHelper` 接口的实现，提供 `ShouldSaveAudit`，`CreateAuditInfo`，`Save`等方法，如下图：

![QQ截图20190727152907.png](/img/QQ截图20190727152907.png)

`IAuditingConfiguration`：定义用于配置审计的接口；

`AuditingConfiguration`：`IAuditingConfiguration` 的默认实现，如下图：

![QQ截图20190727151604.png](/img/QQ截图20190727151604.png)

![QQ截图20190727153811.png](/img/QQ截图20190727153811.png)

`AbpKernelModule` 的预初始化方法 `PreInitialize` 中调用，如下图：

![QQ截图20190727151740.png](/img/QQ截图20190727151740.png)

`IAuditingSelectorList`：用于选择要审计的类/接口的选择器函数列表。

`AuditingSelectorList`：`IAuditingSelectorList`接口的默认实现；

`NamedTypeSelector`：类型选择器，这个对象的核心属性是一个以Type为输入参数，返回Bool类型的委托 Predicate；

![QQ截图20190727153936.png](/img/QQ截图20190727153936.png)

参考：

[ABP源码分析十九：Auditing](https://www.cnblogs.com/1zhk/p/5342947.html)