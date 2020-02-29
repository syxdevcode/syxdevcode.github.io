---
title: ABP框架学习记录（20）- Services的实现
date: 2019-08-29 11:35:56
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（20）- Services的实现

`Services` 的实现在ABP项目的目录位置：

![QQ截图20190829113722.png](/img/QQ截图20190829113722.png)

`AbpServiceBase`：可以作为服务的基类，它提供一些有用的对象属性注入和一些基本方法；

![QQ截图20190903095122.png](/img/QQ截图20190903095122.png)

`IApplicationService`：所有应用程序服务都必须实现此接口，以便按惯例识别它们。

应用：

![QQ截图20190909111242.png](/img/QQ截图20190909111242.png)

`IAvoidDuplicateCrossCuttingConcerns`：避免重复的交叉问题

![QQ截图20190903103311.png](/img/QQ截图20190903103311.png)

`ApplicationService`：服务基类；

![QQ截图20190903103449.png](/img/QQ截图20190903103449.png)

![QQ截图20190903103642.png](/img/QQ截图20190903103642.png)

`WebApi` 项目，在动态生成服务的功能中，通过反射，获取到 `ApplicationService` 作为基类的项：

![QQ截图20190905140646.png](/img/QQ截图20190905140646.png)

`CrudAppServiceBase`：为 `CrudAppService` 和 `AsyncCrudAppService` 提供基类，使用的时候也是这两个类；

基类提供分页，排序，对象映射，鉴权等方法；

![QQ截图20190906143117.png](/img/QQ截图20190906143117.png)

![QQ截图20190906151327.png](/img/QQ截图20190906151327.png)

`ICrudAppService`：提供 同步 CRUD 操作的服务接口；

![QQ截图20190906151648.png](/img/QQ截图20190906151648.png)

`CrudAppService`：继承 `CrudAppServiceBase` 抽象类和 `ICrudAppService` 泛型接口；

![QQ截图20190906152516.png](/img/QQ截图20190906152516.png)

其他泛型抽象类定义：

![QQ截图20190906153453.png](/img/QQ截图20190906153453.png)

`IAsyncCrudAppService`：提供 异步 CRUD 操作的服务接口；

![QQ截图20190906151604.png](/img/QQ截图20190906151604.png)

`AsyncCrudAppService`：继承 `CrudAppServiceBase` 抽象类和 `IAsyncCrudAppService` 泛型接口；

![QQ截图20190906153733.png](/img/QQ截图20190906153733.png)