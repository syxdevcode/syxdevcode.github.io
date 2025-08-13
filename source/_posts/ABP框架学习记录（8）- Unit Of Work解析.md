---
title: ABP框架学习记录（8）- Unit Of Work解析
date: 2019-05-09 13:23:21
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（8）- Unit Of Work解析

目录结构：在ABP项目/Domain/Uow目录下：

![QQ截图20190509132703.png](/img/QQ截图20190509132703.png)

在 `AbpBootstrapper` 类，对拦截器的注册方法中，提供对 `UnitOfWork` 的注册：

![QQ截图20190603160354.png](/img/QQ截图20190603160354.png)

`AbpKernelModule` 类 `PostInitialize` 方法注册默认 `IUnitOfWork` 接口的实现 `NullUnitOfWork`：

![QQ截图20190604141032.png](/img/QQ截图20190604141032.png)

`AbpKernelModule` 注册过滤器：

![QQ截图20190604135212.png](/img/QQ截图20190604135212.png)

## UnitOfWorkRegistrar

`UnitOfWorkRegistrar` ：为Unit Of Work机制注册所需类的拦截器。

`Initialize` 初始化方法：将 `UnitOfWorkInterceptor` 拦截器添加到标注了 `UnitOfWorkAttribute` 特性方法的类，以及实现`IRepository` 或 `IApplicationService` 的类上。

![QQ截图20190603160828.png](/img/QQ截图20190603160828.png)

`HandleTypesWithUnitOfWorkAttribute` 和 `HandleConventionalUnitOfWorkTypes` 方法：

![QQ截图20190603161027.png](/img/QQ截图20190603161027.png)

应用拦截器：

![QQ截图20190611151213.png](/img/QQ截图20190611151213.png)

## UnitOfWorkDefaultOptions

`UnitOfWorkDefaultOptions/IUnitOfWorkDefaultOptions`：配置选项，内部类型或成员只能在同一程序集的文件中访问。

`UnitOfWorkDefaultOptions` 提供 `RegisterFilter` 方法来注册 过滤器：

![QQ截图20190611145011.png](/img/QQ截图20190611145011.png)

`DataFilterConfiguration`：数据过滤器配置类：

![QQ截图20190611145513.png](/img/QQ截图20190611145513.png)

`DefaultConnectionStringResolver`：实现 `IConnectionStringResolver` 接口，获取连接数据库字符串；

## UnitOfWorkInterceptor

`UnitOfWorkInterceptor` 继承 `Castle.DynamicProxy.IInterceptor` 接口，用于管理数据库连接和事务。

通过构造注入 `IUnitOfWorkManager` 和 `IUnitOfWorkDefaultOptions`：

![QQ截图20190603164632.png](/img/QQ截图20190603164632.png)

实现 `IInterceptor` 接口 `Intercept` 方法，通过注入 `IUnitOfWorkDefaultOptions` 接口，获取到 `UnitOfWorkDefaultOptions` 类的实例，并通过 `UnitOfWorkDefaultOptions` 的扩展方法，获取到使用 `UnitOfWorkAttribute` 特性标注的 类，接口和方法：

`Intercept` 方法：应用拦截器，

![QQ截图20190611151458.png](/img/QQ截图20190611151458.png)

使用 `UnitOfWorkDefaultOptionsExtensions` 扩展类，查询应用了 `UnitOfWorkAttribute` 特性的方法/类；

![QQ截图20190611151724.png](/img/QQ截图20190611151724.png)

`UnitOfWorkDefaultOptions` 的扩展类 `UnitOfWorkDefaultOptionsExtensions` ：

![QQ截图20190604111433.png](/img/QQ截图20190604111433.png)

## UnitOfWorkAttribute

`UnitOfWorkAttribute`：用于指示声明方法是原子的，应该被视为一个工作单元。拦截具有此属性的方法，打开数据库连接并在调用方法之前启动事务。在方法调用结束时，如果没有异常，则提交事务并将所有更改应用于数据库，否则它会回滚。如果在调用此方法之前已有工作单元，则此属性无效，如果是，则使用相同的事务。

![QQ截图20190611152126.png](/img/QQ截图20190611152126.png)

## UnitOfWork 事务

基于接口隔离原则的考量，ABP作者将UnitOfWork的方法分到了三个不同的接口中。

`IUnitOfWorkCompleteHandle`:定义了UOW同步和异步的complete方法。实现UOW完成时候的逻辑。

`IActiveUnitOfWork`：一个UOW除了以上两个接口中定义的方法和属性外，其他的属性和方法都在这个接口定义的。比如Completed，Disposed，Failed事件代理，Filter的enable和disable,以及同步、异步的SaveChanges方法。

`IUnitOfWork`：继承了上面两个接口。定义了外层的IUnitOfWork的引用和UOW的begin方法。 ABP是通过构建一个UnitOfWork的链，将不同的方法纳入到一个事务中。

### Unit Of Work运行流程

* 1，UOW拦截器被注入到需要UOW的类中。
* 2，ABP执行标注了UnitOfWork特性的方法时。UOW拦截器以begin()->realAction()->complete()->dispose()顺序执行，其中realAction是被调用的真实业务操作。 UOW拦截器是通过using这种方式调用IUnitOfWork的某个具体实现，这就确保begin 和 dispose也总是会被执行的。 这里需要注意complete却不一定会被执行，比如在complete方法被调用前方法的执行产生了异常。
* 3，当执行一连串的操作时（A方法->B方法->C方法，假设这三个方法都标注了UnitOfWork特性），ABP在执行A方法前会创建整个过程中唯一的IUnitOfWork对象，该对象会启动.NET事务。在执行到B，C方法只会创建InnerUnitOfWorkCompleteHandle。 InnerUnitOfWorkCompleteHandle与IUnitOfWork对象的差异在于它不会创建真实的事务。但ABP会调用其Complete，以告知ABP其对应的方法以成功完成，可以提交事务。
* 4，事务可以回滚的关键关键在于IUnitOfWork对象在被Dispose时候会检查Complete方法有没有被执行，没有的话就认为这个UOW标注的方法没有顺利完成，从而导致事务的回滚操作。
* 5，整个事务的提交是通过第一个UOW（也是唯一个）的Complete方法执行时提交的。

### UnitOfWorkBase

`UnitOfWorkBase` ： 继承 `IUnitOfWork` 的抽象类，真正实现事务控制的方法是由这个抽象类的子类实现的（比如，真正创建 `TransactionScope` 的操作是在 `EfUnitOfWork` ，`NhUnitOfWork` 这样的之类中实现的）。UOW中除了事务控制逻辑以外的逻辑都是由 `UnitOfWorkBase` 抽象类实现的。 

ABP中共有以下4个具体的UOW类型，他们都继承自 `UnitOfWorkBase`。`Entity Framework`, `Nhibernate`。

构造函数：

![QQ截图20190604141528.png](/img/QQ截图20190604141528.png)

启用/禁用 filter：

![QQ截图20190604143018.png](/img/QQ截图20190604143018.png)

提供 `Completed`，`Disposed`，`Failed` 事件：

![QQ截图20190604143152.png](/img/QQ截图20190604143152.png)

![QQ截图20190604143424.png](/img/QQ截图20190604143424.png)

`Begin`，`SaveChanges`，`SaveChangesAsync`：

![QQ截图20190611161819.png](/img/QQ截图20190611161819.png)

### UnitOfWorkManager

工作单元管理类；对外提供UOW的功能(用于创建UnitOfWork，并开启UnitOfWork流程)，对内调用各种UOW功能的各种组件。

![QQ截图20190611162244.png](/img/QQ截图20190611162244.png)

启用工作单元：

```cs
public IUnitOfWorkCompleteHandle Begin(UnitOfWorkOptions options)
{
    options.FillDefaultsForNonProvidedOptions(_defaultOptions);

    var outerUow = _currentUnitOfWorkProvider.Current;

    /// 如果启用 工作单元事务，且外部 Uow 不为空，则使用 InnerUnitOfWorkCompleteHandle
    /// 对象。
    if (options.Scope == TransactionScopeOption.Required && outerUow != null)
    {
        return new InnerUnitOfWorkCompleteHandle();
    }

    var uow = _iocResolver.Resolve<IUnitOfWork>();

    uow.Completed += (sender, args) =>
    {
        _currentUnitOfWorkProvider.Current = null;
    };

    uow.Failed += (sender, args) =>
    {
        _currentUnitOfWorkProvider.Current = null;
    };

    uow.Disposed += (sender, args) =>
    {
        _iocResolver.Release(uow);
    };

    //Inherit filters from outer UOW
    if (outerUow != null)
    {
        options.FillOuterUowFiltersForNonProvidedOptions(outerUow.Filters.ToList());
    }

    uow.Begin(options);

    //Inherit tenant from outer UOW
    if (outerUow != null)
    {
        uow.SetTenantId(outerUow.GetTenantId(), false);
    }

    _currentUnitOfWorkProvider.Current = uow;

    return uow;
}
```

EF Core 启用事务：

![QQ截图20190611164826.png](/img/QQ截图20190611164826.png)

`UnitOfWork` 拦截器调用 `UnitOfWorkManager` 开启UOW流程的代码。

![QQ截图20190611162510.png](/img/QQ截图20190611162510.png)

### InnerUnitOfWorkCompleteHandle

`InnerUnitOfWorkCompleteHandle`：实现 `IUnitOfWorkCompleteHandle`， 用户工作范围的内联单元，内部工作单元实际使用外部工作单元，并且对 `IUnitOfWorkCompleteHandle.Complete` 调用没有影响。但是如果没有调用它，则会在UOW结束时抛出异常以回滚UOW。

![QQ截图20190611163052.png](/img/QQ截图20190611163052.png)

提交事务：

![QQ截图20190611165617.png](/img/QQ截图20190611165617.png)

### AsyncLocalCurrentUnitOfWorkProvider

`AsyncLocalCurrentUnitOfWorkProvider`：实现 `ICurrentUnitOfWorkProvider` 接口，获取或设置当前 `IUnitOfWork` 实例；

### IActiveUnitOfWork

`IActiveUnitOfWork`：用于处理活动的工作单元，此接口不能被注入，可以使用 IUnitOfWorkManager 替换。

```cs
/// <summary>
/// Gets current unit of work.
/// </summary>
protected IActiveUnitOfWork CurrentUnitOfWork { get { return UnitOfWorkManager.Current; } }
```

参考：

[ABP源码分析十：Unit Of Work](http://www.cnblogs.com/1zhk/p/5309043.html)