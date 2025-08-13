---
title: ASP.NET MVC 页面生命周期
date: 2019-02-19 15:10:11
tags:
- DotNet
- MVC/Web API
- ASP.NET/IIS请求处理管道
categories: 
- MVC/Web API
---
# ASP.NET MVC 页面生命周期

整个ASP.NET MVC框架是通过自定义的 `HttpModule` 和 `HttpHandler` 对ASP.NET 进行扩展构建起来的。

* 自定义的 `HttpModule` : `UrlRoutingModule`
* 自定义的 `HttpHandler`: `MvcHandler`

![1458860-5d9e4724f9e32ccb.png](/img/1458860-5d9e4724f9e32ccb.png)

![190104561291113.png](/img/190104561291113.png)

## HttpApplication与HttpModule

HTTP请求由ASP.NET运行时接管之后，`HttpRuntime` 会利用 `HttpApplicationFactory` 创建或从 `HttpApplication`对象池中取出一个`HttpApplication`对象，同时ASP.NET会根据配置文件来初始化注册的`HttpModule`，`HttpModule`在初始化时会订阅`HttpApplication`中的事件来实现对HTTP请求的处理。

![190047183794497.png](/img/190047183794497.png)

<font color=#ff0000 size=4 face="黑体">ASP.NET MVC的入口在`UrlRoutingModule`,它订阅了`HttpApplication`的第7个管道事件`PostResolveRequestCahce`，并且在`HtttpApplication`的第7个管道事件处对请求进行了拦截。</font>

## UrlRoutingModule

通过在全局Web.Config中注册 `System.Web.Routing.UrlRoutingModule`，IIS请求处理管道接到请求后，就会加载 `UrlRoutingModule`类型的Init()方法，第7个管道事件 `PostResolveRequestCahce` 对请求进行了拦截。

当请求到达 `UrlRoutingModule` 的时候，`UrlRoutingModule` 取出请求中的 Controller、Action等RouteData信息，与路由表中的所有规则进行匹配，若匹配，把请求交给 `IRouteHandler`，即 `MVCRouteHandler`。

```cs
//通过RouteCollection的静态方法GetRouteData获取到封装路由信息的RouteData实例
RouteData routeData = this.RouteCollection.GetRouteData(context);

//再从RouteData中获取MVCRouteHandler
IRouteHandler routeHandler = routeData.RouteHandler;
```

## MVCRouteHadnler

`MvcRouteHandler` 在HttpApplication的第一个管道事件，使用 `MapRoute()` 方法注册路由的时候，通过`Route`类的构造函数把`MVCRouteHandler`注入到路由中。

```cs
public static Route MapRoute(this RouteCollection routes, string name, string url, object defaults, object constraints, string[] namespaces)
{
    if (routes == null) {
        throw new ArgumentNullException("routes");
    }
    if (url == null) {
        throw new ArgumentNullException("url");
    }
    // MvcRouteHandler
    Route route = new Route(url, new MvcRouteHandler()) {
        Defaults = new RouteValueDictionary(defaults),
        Constraints = new RouteValueDictionary(constraints),
        DataTokens = new RouteValueDictionary()
    };

    if ((namespaces != null) && (namespaces.Length > 0)) {
        route.DataTokens["Namespaces"] = namespaces;
    }

    routes.Add(name, route);

    return route;
}
```

`MVCRouteHandler`是用来生成实现`IHttpHandler`接口的`MvcHandler`。

```cs
namespace System.Web.Routing
{  
    public interface IRouteHandler
    {
        IHttpHandler GetHttpHandler(RequestContext requestContext);
    }
}
```

## MvcHandler

第7个事件生成了 `MvcHandler`。

第11和第12个事件之间调用了`MvcHandler`的`ProcessRequest`方法。

通过`RouteHandler`的 `GetHttpHandler()`方法获取到实现了`IHttpHandler`接口的`MVCHandler`。

```cs
IHttpHandler httpHandler = routeHandler.GetHttpHandler(requestContext);
context.RemapHandler(httpHandler);
```

`MvcHandler` 部分源码:

```cs
public class MvcHandler : IHttpAsyncHandler, IHttpHandler, IRequiresSessionState
{
    protected internal virtual void ProcessRequest(HttpContextBase httpContext)
    {
        SecurityUtil.ProcessInApplicationTrust(() =>
        {
            IController controller;
            IControllerFactory factory;
            ProcessRequestInit(httpContext, out controller, out factory);//初始化了ControllerFactory
            try
            {
                controller.Execute(RequestContext);
            }
            finally
            {
                factory.ReleaseController(controller);
            }
        });
    }

     private void ProcessRequestInit(HttpContextBase httpContext, out IController controller, out IControllerFactory factory) {
        bool? isRequestValidationEnabled = ValidationUtility.IsValidationEnabled(HttpContext.Current);
        if (isRequestValidationEnabled == true) {
            ValidationUtility.EnableDynamicValidation(HttpContext.Current);
        }
        AddVersionHeader(httpContext);
        RemoveOptionalRoutingParameters();
        string controllerName = RequestContext.RouteData.GetRequiredString("controller");
        factory = ControllerBuilder.GetControllerFactory();
        controller = factory.CreateController(RequestContext, controllerName);
        if (controller == null) {
          throw new InvalidOperationException(String.Format(CultureInfo.CurrentCulture,MvcResources.ControllerBuilder_FactoryReturnedNull,factory.GetType(),controllerName));
        }
    }
}
```

## ControllerFactory

通过`ControllerBuilder`的静态方法`GetControllerFactory`获取到实现`IControllerFactory`接口的`ControllerFactory`。通过`ControllerFactory`来创建指定名称的控制器，最后将控制器作为out参数传递出去。

## ActionInvoker

ActionInvoker实现了IActionInvoker接口：

```cs
public interface IActionInvoker
{
  bool InvokeAction(ControllerContext controllerContext, string actionName);
}
```

MVC默认的ActionInvoker是ControllerActionInvoker。

ActionInvoker在执行InvokeAction()方法时会需要有关Controller和Action的相关信息，实际上，Controller信息(比如Controller的名称、类型、包含的Action等)被封装在ControllerDescriptor这个类中，Action信息(比如Action的名称、参数、属性、过滤器等)被封装在ActionDescriptor中。ActionDescriptor还提供了一个FindAction()方法，用来找到需要被执行的Action。

## Filters

Filter->Action->Filter->Result

在ASP.NET MVC5中有常用的过滤器有5个：`IAuthenticationFilter`、`IAuthorizationFilter`、`IActionFilter`、`IResultFilter`、`IExceptionFilter`。
在ASP.NET MVC中所有的过滤器最终都会被封装为Filter对象，该对象中FilterScope类型的属性Scope和int类型属性Order用于决定过滤器执行的先后顺序，具体规则如下：

* Order和FilterScope的数值越小，过滤器的执行优先级越高；
* Order比FilterScope具有更高的优先级，在Order属性值相同时FilterScope才会被考虑

```cs
//数值越小，执行优先级越高
public enum FilterScope
{
    Action= 30,
    Controller= 20,
    First= 0,
    Global= 10,
    Last= 100
}
```

## ActionResult

ActionResult是一个抽象类：

```cs
public abstract class ActionResult
{
  public abstract void ExecuteResult(ControllerContext context);
}
```

如果ActionResult是非ViewResult，比如JsonResult, ContentResult，这些内容将直接被输送到Response响应流中，显示给客户端；如果是ViewResult，就会进入下一个渲染视图环节。

## ViewEngine

默认的有`Razor View Engine`和`Web Form View Engine`，实现IViewEngine接口。

IViewEngine接口方法：

* FindPartialView
* FindView
* ReleaseView

参考：

[ASP.NET MVC5请求管道和生命周期](https://www.cnblogs.com/Cwj-XFH/p/6752177.html)

[ASP.Net请求处理机制初步探索之旅 - Part 5 ASP.Net MVC请求处理流程](http://www.cnblogs.com/edisonchou/p/4226558.html)

[ASP.NET MVC请求处理管道生命周期的19个关键环节(13-19)](http://www.cnblogs.com/darrenji/p/3795690.html)

[How ASP.NET MVC Works?](http://www.cnblogs.com/artech/archive/2012/04/10/how-mvc-works.html)

[ASP.NET MVC中的ActionFilter是如何执行的？](http://www.cnblogs.com/artech/archive/2012/08/06/action-filter.html)

[认识ASP.NET MVC的5种AuthorizationFilter](http://www.cnblogs.com/artech/archive/2012/07/02/AuthorizationFilter.html)

[ASP.NET MVC的View是如何被呈现出来的？[设计篇]](http://www.cnblogs.com/artech/archive/2012/08/22/view-engine-01.html)

[ASP.NET MVC的View是如何呈现出来的[实例篇]](http://www.cnblogs.com/artech/archive/2012/08/23/view-engine-02.html)