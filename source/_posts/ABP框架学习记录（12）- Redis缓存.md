---
title: ABP框架学习记录（12）- Redis缓存
date: 2019-07-01 14:34:59
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（12）- Redis缓存

在 ABP 框架中，单独新建了一个项目来实现 Redis 缓存，具体项目/目录如下图：

![QQ截图20190701143752.png](/img/QQ截图20190701143752.png)

首先，`AbpRedisCacheModule` 模块用于使用Redis服务器取代ABP的缓存系统，`AbpRedisCacheModule` 依赖于 `AbpKernelModule`；

![QQ截图20190701175116.png](/img/QQ截图20190701175116.png)

`AbpRedisCacheOptions` : 提供 `AbpStartupConfiguration` 属性，为 `RedisCacheConfigurationExtensions` 扩展类提供 `IIocManager` 的实例： 
![QQ截图20190701175312.png](/img/QQ截图20190701175312.png)

提供 `IIocManager` 实例，并注册 `AbpRedisCacheManager` 类做为 `ICacheManager` 的默认实现：

![QQ截图20190701180120.png](/img/QQ截图20190701180120.png)

`AbpRedisCacheManager` : 用户管理 `AbpRedisCache`;

![QQ截图20190701180537.png](/img/QQ截图20190701180537.png)

`IAbpRedisCacheDatabaseProvider/AbpRedisCacheDatabaseProvider` ：用户获取 `IDatabase` 实例；

![QQ截图20190701181130.png](/img/QQ截图20190701181130.png)

`IRedisCacheSerializer/DefaultRedisCacheSerializer`：持久化和检索时使用的所有自定义（反）序列化方法；

![QQ截图20190701181427.png](/img/QQ截图20190701181427.png)

`RedisDatabaseExtensions`：扩展类；

![QQ截图20190701181522.png](/img/QQ截图20190701181522.png)