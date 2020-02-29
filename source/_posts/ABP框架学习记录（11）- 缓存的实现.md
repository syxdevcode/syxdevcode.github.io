---
title: ABP框架学习记录（11）- 缓存的实现
date: 2019-06-24 14:17:55
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（11）- 缓存的实现

缓存在 ABP 项目中的路径 `ABP/Runtime`：

![QQ截图20190628105843.png](/img/QQ截图20190628105843.png)

ABP中实现两种 `Cache` ：`MemroyCache` 和 `RedisCache`。两者都继承至 `ICache` 接口（准确说是 `CacheBase` 抽象类）。ABP核心模块封装了 `MemroyCache` 来实现ABP中的默认缓存功能。 `Abp.RedisCache` 这个模块封装 `RedisCache` 来实现缓存（通过 `StackExchange.Redis` 这个类库访问redis）

`ICache` 接口：定义可以按键存储和获取项目的缓存。

![QQ截图20190628151914.png](/img/QQ截图20190628151914.png)

`ICache` 接口中的方法 `Get/GetAsync` 有两个参数：
key：缓存关键字,字符串类型；
工厂：没有找到指定key的缓存条目时调用传入的action来创建cache。工厂方法应该创建并返回实际的条目。如果给定的key在缓存中找到了，那么不会调用该action。

![QQ截图20190628152326.png](/img/QQ截图20190628152326.png)

`CacheBase`：提供继承 `ICache` 接口的抽象类；

![QQ截图20190701132308.png](/img/QQ截图20190701132308.png)

`CacheExtensions`：提供 `ICache`的扩展类：

![QQ截图20190701132333.png](/img/QQ截图20190701132333.png)

`ICacheManager/CacheManagerBase`：提供缓存管理器；

![QQ截图20190628162728.png](/img/QQ截图20190628162728.png)

`ICacheManager.GetCache` 方法返回一个ICache。第一次请求时会创建缓存，并通过 `CachingConfiguration` 中的`CacheConfigurator` 完成对该Cache的配置，以后都是返回相同的缓存对象。因此，我们可以在不同的类（客户端）中共享具有相同名字的相同缓存。

![QQ截图20190628162758.png](/img/QQ截图20190628162758.png)

`ICachingConfiguration/CachingConfiguration`：用于配置 缓存系统，首先通过在 `AbpKernelModule` 类，`PreInitialize` 方法配置缓存：

![QQ截图20190701142545.png](/img/QQ截图20190701142545.png)

定义：

![QQ截图20190701141638.png](/img/QQ截图20190701141638.png)

![QQ截图20190701141747.png](/img/QQ截图20190701141747.png)

`ICacheConfigurator/CacheConfigurator`：表示已注册的缓存配置器,定义两个构造函数，其中的参数 `Action<ICache> initAction` 表示创建的回调函数；

![QQ截图20190701142143.png](/img/QQ截图20190701142143.png)

在 `AbpKernelModule` 类，`PreInitialize` 方法调用：

![QQ截图20190701142545.png](/img/QQ截图20190701142545.png)

`CacheManagerExtensions`：提供对 `CacheManager` 扩展类，提供 `GetCache<TKey, TValue>` 方法，具体实现调用 `CacheExtensions` 的 `AsTyped<TKey, TValue>` 扩展方法；

![QQ截图20190701132812.png](/img/QQ截图20190701132812.png)

![QQ截图20190701133029.png](/img/QQ截图20190701133029.png)

`AsTyped<TKey, TValue>` 扩展方法，实例化了一个 `TypedCacheWrapper<TKey, TValue>(cache)` 类型的对象，并将 `CacheManagerExtensions` 中扩展方法 `GetCache<TKey, TValue>` 的 `TKey, TValue`，然后继续传递给  `CacheExtensions` 中的扩展方法 `AsTyped<TKey, TValue>` 的 `TKey, TValue`，最终，在 `TypedCacheWrapper` 的泛型类中接收，并为其自身的方法提供 `TKey, TValue`。

![QQ截图20190701133134.png](/img/QQ截图20190701133134.png)

`ITypedCache/TypedCacheWrapper/TypedCacheExtensions`：支持泛型key和value的缓存接口与实现，其内部通过封装ICache实例和`CacheExtension`定义的对ICache的扩展方法来是实现泛型版本的 `Icache`。

![QQ截图20190628153126.png](/img/QQ截图20190628153126.png)

![QQ截图20190628153232.png](/img/QQ截图20190628153232.png)

`AbpMemoryCacheManager` ：重写了 `CacheManagerBase` 的 `CreateCacheImplementation` 方法，该方法用于创建真实的 `Icache` 对象。 具体到 `AbpMemoryCacheManager` 就是创建 `AbpMemoryCache`。

![QQ截图20190701143053.png](/img/QQ截图20190701143053.png)

`AbpMemoryCache`：通过CLR的 `MemoryCache` 来实现 `Icache` 。

![QQ截图20190701143151.png](/img/QQ截图20190701143151.png)

**使用**

![QQ截图20190701172446.png](/img/QQ截图20190701172446.png)

参考：

[ABP源码分析十三：缓存Cache实现](https://www.cnblogs.com/1zhk/p/5328547.html)