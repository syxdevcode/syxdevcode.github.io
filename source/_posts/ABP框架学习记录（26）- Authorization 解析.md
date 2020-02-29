---
title: ABP框架学习记录（26）- Authorization 解析
date: 2019-10-17 15:22:11
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（26）- Authorization 解析

本文 以 `Volo.Abp` 解决方案展开研究；

在 `Volo.Abp` 解决方案中，提供了单独的项目 实现 `Authorization` 功能，如下图:

![QQ截图20191017152520.png](/img/QQ截图20191017152520.png)

`Authorization`：中文解释，授权。ABP 中检查是否对 `用户或者角色` 授予权限。整个功能可以分为两个方面理解：
1，许可定义；
2，检查许可；

## Permission

`Permission` 的中文解释为：许可，允许。

首先介绍 `Permission` 相关的类定义：

`PermissionDefinition`：许可定义，包括父级权限和子级列表。

![QQ截图20191021114332.png](/img/QQ截图20191021114332.png)

`PermissionGroupDefinition`：定义许可组；

![QQ截图20191021131218.png](/img/QQ截图20191021131218.png)

`PermissionGrantResult`：授权结果；

![QQ截图20191021131648.png](/img/QQ截图20191021131648.png)

`IPermissionChecker`：检查授权，提供 `IsGrantedAsync` 方法检查是否有权限。

![QQ截图20191021104232.png](/img/QQ截图20191021104232.png)

`PermissionChecker`：实现 `IPermissionChecker` 接口；

![QQ截图20191021131755.png](/img/QQ截图20191021131755.png)

`IPermissionDefinitionContext`：许可定义上下文，封装了 `PermissionGroupDefinition` 对象，并且提供了添加，移除的方法；

![QQ截图20191021132317.png](/img/QQ截图20191021132317.png)

`PermissionDefinitionContext`：实现 `IPermissionDefinitionContext` 接口；

![QQ截图20191021132414.png](/img/QQ截图20191021132414.png)

`IPermissionDefinitionManager`：管理 `PermissionDefinition`;

![QQ截图20191021135949.png](/img/QQ截图20191021135949.png)

`PermissionDefinitionManager`：默认实现 `IPermissionDefinitionManager` 接口。

![QQ截图20191021140150.png](/img/QQ截图20191021140150.png)

`IPermissionDefinitionProvider`：定义 `PermissionDefinition` 提供者接口；对 `Define` 方法传入的 `IPermissionDefinitionContext` 对象进行操作；

![QQ截图20191021140648.png](/img/QQ截图20191021140648.png)

`PermissionDefinitionProvider`：实现 `IPermissionDefinitionProvider` 接口；

![QQ截图20191021140742.png](/img/QQ截图20191021140742.png)

`IPermissionStore`：定义许可 存储接口；

![QQ截图20191021141034.png](/img/QQ截图20191021141034.png)

`NullPermissionStore` ：`IPermissionStore` 接口的空实现。

`IPermissionValueProvider`：定义获取 `PermissionValue` 的接口；

![QQ截图20191021151209.png](/img/QQ截图20191021151209.png)

`PermissionValueProvider`：实现 `IPermissionValueProvider` 接口；

![QQ截图20191021171013.png](/img/QQ截图20191021171013.png)

`RolePermissionValueProvider`：提供角色相关许可；

`UserPermissionValueProvider`：提供用户相关许可；

`ClientPermissionValueProvider`：提供客户端相关许可；

`IPermissionValueProviderManager`：定义管理 `IPermissionValueProvider` 的接口；

![QQ截图20191021171140.png](/img/QQ截图20191021171140.png)

`PermissionValueProviderManager`：实现 `IPermissionValueProviderManager` 接口；

![QQ截图20191021171437.png](/img/QQ截图20191021171437.png)

`PermissionOptions`：许可选项；

![QQ截图20191021141758.png](/img/QQ截图20191021141758.png)

`PermissionValueCheckContext`：检查 `PermissionValue` 的上下文；

![QQ截图20191021171820.png](/img/QQ截图20191021171820.png)

## AbpAuthorizationModule

这一部分主要分析 `AbpAuthorizationModule` 的实现。

在 `Volo.Abp` 解决方案中，其 `Module` 系统集成 Core 自带的 `IServiceCollection` ，通过自定义的 `ServiceConfigurationContext` 类，封装了 `IServiceCollection` 对象：

![QQ截图20191021175125.png](/img/QQ截图20191021175125.png)

`AbpAuthorizationModule`：定义`Module`，`OnRegistred` 扩展方法 添加 拦截器注册者；

![QQ截图20191021174716.png](/img/QQ截图20191021174716.png)

### PreConfigureServices

`AuthorizationInterceptorRegistrar`：拦截器注册类；

![QQ截图20191021175737.png](/img/QQ截图20191021175737.png)

`AuthorizationInterceptor`：定义拦截器；

![QQ截图20191021175839.png](/img/QQ截图20191021175839.png)

`MethodInvocationAuthorizationContext`：定义方法调用 许可上下文，校验方法是否被允许；

![QQ截图20191021180118.png](/img/QQ截图20191021180118.png)

`IMethodInvocationAuthorizationService`：定义校验方法许可的接口；

`MethodInvocationAuthorizationService`：实现 `IMethodInvocationAuthorizationService` 接口；

![QQ截图20191021180541.png](/img/QQ截图20191021180541.png)

### ConfigureServices

![QQ截图20191021180851.png](/img/QQ截图20191021180851.png)

`PermissionRequirement`：许可参数；

![QQ截图20191021181401.png](/img/QQ截图20191021181401.png)

`PermissionRequirementHandler`：定义 回调 `Handler`；

![QQ截图20191021181600.png](/img/QQ截图20191021181600.png)

## 其它

`IAbpAuthorizationService`：定义 `Abp` 授权服务接口；继承 Dot Net Core自带 `IAuthorizationService` 接口；

`AbpAuthorizationService`：实现  `IAuthorizationService` 接口;

![QQ截图20191021182258.png](/img/QQ截图20191021182258.png)

`IAbpAuthorizationPolicyProvider`：定义 授权 策略 提供者接口；继承 Dot Net Core 自带 `IAuthorizationPolicyProvider` 接口；

`AbpAuthorizationPolicyProvider`：实现 `IAbpAuthorizationPolicyProvider` 接口；

![QQ截图20191021182527.png](/img/QQ截图20191021182527.png)

