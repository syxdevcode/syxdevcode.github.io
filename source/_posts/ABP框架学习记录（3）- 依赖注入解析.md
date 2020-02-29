---
title: ABP框架学习记录（3）- 依赖注入解析
date: 2019-04-30 17:04:33
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（3）- 依赖注入解析

ABP的依赖注入的实现本质上是依赖于Castle依赖注入的框架。
实现途径有两种：一种实现途径是通过实现`IConventionalDependencyRegistrar` 的实例定义注入的约定（规则），然后通过 `IocManager` 来读取这个规则完成依赖注入。另一种实现途径是直接 `IocManager` 的 `Register` 方法直接完成注入。

目录结构：

代码在Abp项目文件的Dependency文件夹：

![QQ截图20190430174143.png](/img/QQ截图20190430174143.png)

## 注入方式

### 直接注入

Abp定义 `IocManager` 类，以提供依赖注入的管理，内部提供 `IocManager` 静态字段和 `IocContainer` 实例字段。

`IocManager` 类定义了静态构造和实例构造，以确保在创建第一个实例或引用任何静态成员之前自动调用静态构造函数。

![QQ截图20190505153836.png](/img/QQ截图20190505153836.png)

提供Register方法：

![QQ截图20190505164520.png](/img/QQ截图20190505164520.png)

`AbpModule` 有个受保护的 `IocManager` 的成员，所以 `AbpModule` 的派生类都可以使用这个 `IocManager` 完成注册。

![QQ截图20190505154320.png](/img/QQ截图20190505154320.png)

### 约定（规则）

`ConventionalRegistrationConfig`：封装了一个bool属性 `InstallInstallers`，用以告诉Abp底层框架是否要register相应assembly中的通过IWindsorInstaller接口指定的register规则，默认true。

`IConventionalRegistrationContext`/`ConventionalRegistrationContext`: 和其他上下文类起的作用类似。主要就是作为方法参数方便方法间的传递数据。这里主要封装了 `Assembly` ，`IocManager` 和 `ConventionalRegistrationConfig`。

`IConventionalDependencyRegistrar`：用于按约定注册依赖项。

`IocManager` 提供 `IConventionalDependencyRegistrar` 的集合，`IocManager` 在 `RegisterAssemblyByConvention` 方法中遍历这个list，并根据`IConventionalDependencyRegistrar` 的实例中定义的规则来完成register。

```cs
private readonly List<IConventionalDependencyRegistrar> _conventionalRegistrars;
```

![QQ截图20190505163142.png](/img/QQ截图20190505163142.png)

`IConventionalDependencyRegistrar` 接口实现：

![QQ截图20190506163107.png](/img/QQ截图20190506163107.png)

![QQ截图20190505163519.png](/img/QQ截图20190505163519.png)

## 扩展方法

ABP提供 `IocRegistrarExtensions`,`IocResolverExtensions` 来扩展 `IIocRegistrar`，`IIocResolver` 接口的实现类。

IocRegistrarExtensions：

![QQ截图20190505165639.png](/img/QQ截图20190505165639.png)

IocResolverExtensions：

![QQ截图20190505170652.png](/img/QQ截图20190505170652.png)

`IDisposableDependencyObjectWrapper<out T>`/`IDisposableDependencyObjectWrapper` 接口：提供释放已解析的对象的方法。

![QQ截图20190505171227.png](/img/QQ截图20190505171227.png)

应用：

![QQ截图20190505170929.png](/img/QQ截图20190505170929.png)

参考：

[ABP源码分析六：依赖注入的实现](http://www.cnblogs.com/1zhk/p/5296124.html)
