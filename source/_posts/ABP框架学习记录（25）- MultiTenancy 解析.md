---
title: ABP框架学习记录（25）- MultiTenancy 解析
date: 2019-09-25 13:48:54
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（25）- MultiTenancy 解析

## 简介

多租户和单租户托管是SaaS公司提供服务的两种方式。多租户托管是指在同一软件实例上存在许多客户端，它们共享基础结构，数据库和/或应用程序服务器。它便宜一些，但是有风险。单租户是指租户不共享任何东西。它更昂贵并且需要更多的管理，因为它需要为每个客户端运行完整的软件堆栈。

软件即服务（SaaS）产品可以具有各种级别的多租户。在应用程序服务器级别，可以有一个具有负载均衡功能的应用程序服务器池，可以为多个客户端提供服务。在数据库级别，每个租户可以有一个数据库，也可以在所有租户之间共享一个数据库。

## 详解

MultiTenancy：多租户

`MultiTenancy` 在ABP项目的目录位置：

![QQ截图20190925151033.png](/img/QQ截图20190925151033.png)

`MultiTenancySides`：标识是租户，租主；

![QQ截图20190925153759.png](/img/QQ截图20190925153759.png)

`MultiTenancySideAttribute`：声明多租户 特性；

![QQ截图20190925162718.png](/img/QQ截图20190925162718.png)

`TenantInfo`：租户信息，只有Id,和租户名称；

![QQ截图20190925155312.png](/img/QQ截图20190925155312.png)

`ITenantStore`：查找租户信息；

`NullTenantStore`：默认实现 `ITenantStore` 接口，

在 `AbpKernelModule` 注册，如果没有注册过，则注册 `NullTenantStore`：

![QQ截图20190925160412.png](/img/QQ截图20190925160412.png)

`Abp.Zero.Common` 项目中，`TenantStore` 实现 `ITenantStore`:

![QQ截图20190925160718.png](/img/QQ截图20190925160718.png)

`TenantResolverCacheItem`：租户获取缓存项

`ITenantResolverCache`：定义获取租户缓存接口；

`NullTenantResolverCache`：`ITenantResolverCache` 接口的空实现；

`Abp.AspNetCore.MultiTenancy` 项目， 在 `HttpContextTenantResolverCache` 类实现 `HttpContextTenantResolverCache` 接口：

![QQ截图20190925162409.png](/img/QQ截图20190925162409.png)

`ITenantResolveContributor`：定义租户解析参与者接口；即实际解析租户的实现类

例如，在 `Abp.AspNetCore.MultiTenancy` 项目的 `DomainTenantResolveContributor` 类：

![QQ截图20190925163152.png](/img/QQ截图20190925163152.png)

`ITenantResolver/TenantResolver`：租户解析者，相当于 `Manager` 的角色；

匹配到真正的 `ITenantResolveContributor` 接口实现：

![QQ截图20190925163736.png](/img/QQ截图20190925163736.png)

![QQ截图20190925163623.png](/img/QQ截图20190925163623.png)

## Abp.Zero.Common

`AbpTenantManager`：管理 租户信息，包括持久化，`Feature`等；

![QQ截图20190925165017.png](/img/QQ截图20190925165017.png)

![QQ截图20190925170710.png](/img/QQ截图20190925170710.png)

![QQ截图20190925170749.png](/img/QQ截图20190925170749.png)

参考：

[Single-tenant vs multi-tenant hosting](https://shareplm.com/single-tenant-vs-multi-tenant-hosting/)