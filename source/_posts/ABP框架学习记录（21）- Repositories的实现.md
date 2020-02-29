---
title: ABP框架学习记录（21）- Repositories的实现
date: 2019-09-11 10:21:58
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（21）- Repositories的实现

`Repositories` 实现在ABP项目的目录位置：

![QQ截图20190911102434.png](/img/QQ截图20190911102434.png)

`IRepository`：定义存储接口，所有的存储库必须实现，即按照约定标识；存储库需要实现泛型版本而不是此接口；

![QQ截图20190911110920.png](/img/QQ截图20190911110920.png)

`IRepositoryOfTEntityAndTPrimaryKey`：泛型存储接口

![QQ截图20190911111150.png](/img/QQ截图20190911111150.png)

![QQ截图20190911132904.png](/img/QQ截图20190911132904.png)

`AbpRepositoryBase`：继承 `IRepository{TEntity,TPrimaryKey}` 泛型接口的泛型抽象类；

![QQ截图20190911133034.png](/img/QQ截图20190911133034.png)

![QQ截图20190911142736.png](/img/QQ截图20190911142736.png)

`AutoRepositoryTypesAttribute`：自动为实体生成 `Repository`；

![QQ截图20190911151234.png](/img/QQ截图20190911151234.png)

![QQ截图20190911151333.png](/img/QQ截图20190911151333.png)

`AbpEntityFrameworkModule` : 解析出 `IEfGenericRepositoryRegistrar`，并应用 `RegisterForDbContext` 方法；

![QQ截图20190911151456.png](/img/QQ截图20190911151456.png)

`EfGenericRepositoryRegistrar`：注册 `Repository`;

![QQ截图20190911151633.png](/img/QQ截图20190911151633.png)

`ISupportsExplicitLoading`：支持需要明确加载的对象；

![QQ截图20190911142957.png](/img/QQ截图20190911142957.png)

![QQ截图20190911105512.png](/img/QQ截图20190911105512.png)

![QQ截图20190911110237.png](/img/QQ截图20190911110237.png)

`RepositoryExtensions`：`Repository` 扩展方法；

![QQ截图20190911154020.png](/img/QQ截图20190911154020.png)