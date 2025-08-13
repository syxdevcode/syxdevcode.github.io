---
title: ABP框架学习记录（19）- Navigation解析
date: 2019-08-28 17:34:49
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（19）- Navigation解析

`Navigation` 目录位置：

![QQ截图20190828173829.png](/img/QQ截图20190828173829.png)

`IHasMenuItemDefinitions`：为具有菜单项的类声明通用的接口；

`MenuDefinition`：实现 `IHasMenuItemDefinitions` 接口，表示应用程序的菜单；

![QQ截图20190828174546.png](/img/QQ截图20190828174546.png)

`MenuItemDefinition`：表示 `MenuDefinition` 类中 `Items` 的项；并且实现 `IHasMenuItemDefinitions` 接口;

`MenuItemDefinition` 中引用 `ILocalizableString`,`IPermissionDependency`,`IFeatureDependency`类型的字段；

![QQ截图20190828175215.png](/img/QQ截图20190828175215.png)

![QQ截图20190829100905.png](/img/QQ截图20190829100905.png)

`INavigationManager`：管理 `MenuDefinition`；

![QQ截图20190829110838.png](/img/QQ截图20190829110838.png)

`NavigationManager`：实现 `INavigationManager` 接口，管理 `MenuDefinition`；

![QQ截图20190829111046.png](/img/QQ截图20190829111046.png)

`INavigationProviderContext`：提供设置导航的基础架构。也就是提供 `INavigationManager` 的实现；

![QQ截图20190829111516.png](/img/QQ截图20190829111516.png)

`NavigationProviderContext`：`INavigationProviderContext` 接口的实现；

![QQ截图20190829111621.png](/img/QQ截图20190829111621.png)

`NavigationProvider`：提供更改 `MenuDefinition` 的抽象类，由更改应用程序导航类实现。

![QQ截图20190829111807.png](/img/QQ截图20190829111807.png)

实现：

![QQ截图20190829112052.png](/img/QQ截图20190829112052.png)

`UserMenu`：表示一个向用户展示的菜单；

![QQ截图20190829112228.png](/img/QQ截图20190829112228.png)

`UserMenuItem`：表示 `UserMenu` 的项；

![QQ截图20190829112710.png](/img/QQ截图20190829112710.png)

`IUserNavigationManager`：定义管理用户菜单接口；

![QQ截图20190829112807.png](/img/QQ截图20190829112807.png)

`UserNavigationManager`：`IUserNavigationManager` 接口的实现，

![QQ截图20190829112944.png](/img/QQ截图20190829112944.png)

使用：

![QQ截图20190829113326.png](/img/QQ截图20190829113326.png)

参考：

[ABP源码分析二十二：Navigation](https://www.cnblogs.com/1zhk/p/5356757.html)