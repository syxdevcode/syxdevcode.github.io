---
title: ASP.NET请求处理管道
date: 2019-02-17 09:49:32
tags:
- DotNet
- ASP.NET/IIS请求处理管道
categories: 
- ASP.NET/IIS请求处理管道
---

![2012071209470854.png](/img/2012071209470854.png)

IIS 6.X和IIS7经典模式，采用aspnet_isapi.dll来响应请求；
IIS7集成模式则执行单独的管道；

注：<font color=#ff0000 size=4 face="黑体">ASP.NET WebForm和ASP.NET MVC 同用一套请求处理管道。</font>

## 创建HttpApplication

创建 `HttpApplication`（即w3wp.exe）对象的时候，先去对象池中查找，没有则创建（编译Global.asax,创建HttpApplication对象），找到则直接返回对象。

HttpApplication的工作包括：

● 初始化的时候加载全部的HttpModule
● 接收请求
● 在不同阶段引发不同的事件，使得HttpModule通过订阅事件的方式加入到请求的处理过程中
● 在一个特定阶段获取一个IHttpHandler实例，最终将请求交给具体的IHttpHandler来实现

`HttpApplication` 创建之后，则是ASP.NET请求处理管道。

### HttpModules & HttpHandlers

** HttpModules**

所有的HttpModules都实现了IHttpModule接口,可以实现 `IHttpModule` 接口自定义 `HttpModule`。

** HttpHandlers**

当某个请求与一个规则匹配后，ASP.NET会调用匹配的HttpHandlerFactory的GetHandler方法来获取一个HttpHandler实例， 最后由一个HttpHandler实例来处理当前请求，生成响应内容

所有的 `HttpHandlers` 都实现了 `IHttpHandler` 接口,可以实现 `IHttpHandler` 接口自定义 `HttpHandler`。

如图：

![190047183794497.png](/img/190047183794497.png)

## Asp.Net请求处理事件

``` cs
1、BeginRequest：HTTP管道开始处理请求时，会触发BeginRequest事件

2、AuthenticateRequest：安全模块对请求进行身份验证时触发该事件

3、PostAuthenticateRequest：安全模块对请求进行身份验证后触发该事件

4、AuthorizeRequest：安全模块对请求进程授权时触发该事件

5、PostAuthorizeRequest：安全模块对请求进程授权后触发该事件

6、ResolveRequestCache：缓存模块利用缓存直接对请求进程响应时触发该事件

7、PostResolveRequestCache：缓存模块利用缓存直接对请求进程响应后触发该事件

8、PostMapRequestHandler：对于访问不同的资源类型，ASP.NET具有不同的HttpHandler对其进程处理。
对于每个请求，ASP.NET会根据扩展名选择匹配相应的HttpHandler类型，成功匹配后触发该事件

9、AcquireRequestState：状态管理模块获取基于当前请求相应的状态(比如SessionState)时触发该事件

10、PostAcquireRequestState：状态管理模块获取基于当前请求相应的状态(比如SessionState)后触发该事件

11、PreRequestHandlerExecute：在实行HttpHandler前触发该事件

12、PostRequestHandlerExecute：在实行HttpHandler后触发该事件

13、ReleaseRequestState：状态管理模块释放基于当前请求相应的状态时触发该事件

14、PostReleaseRequestState：状态管理模块释放基于当前请求相应的状态后触发该事件

15、UpdateRequestCache：缓存模块将HttpHandler处理请求得到的相应保存到输出缓存时触发该事件

16、PostUpdateRequestCache：缓存模块将HttpHandler处理请求得到的相应保存到输出缓存后触发该事件

17、LogRequest：为当前请求进程日志记录时触发该事件

18、PostLogReques：为当前请求进程日志记录后触发该事件

19、EndRequest：整个请求处理完成后触发该事件

20、PreSendRequestHeaders：.Net4.0新增，可以根据发送的Header设置一些参数，
例如，设置一个特殊的Header通知浏览器启用压缩来提高网络传输速度

21、PreSendRequestContent：.Net4.0新增，恰好在 ASP.NET 向客户端发送内容之前发生。
例如，在此事件压缩输出到浏览器的流。
```

<font color=##ff0000 size=4 face="黑体">webform 和 mvc 共用一套管道，在第7个事件 `PostResolveRequestCache` 会根据Url去路由表匹配所有的路由规则，
如果匹配到了路由，则证明请求是MVC程序,反之，则是WebForm。</font>

## WebForm 管道事件摘要

1，第八个事件 `PostMapRequestHandler` 创建Page类对象并转换为IHttpHandler接口。

在这个事件中，对于访问不同的资源类型，ASP.NET具有不同的HttpHandler对其进程处理。对于每个请求，ASP.NET会通过扩展名选择匹配相应的HttpHandler类型，成功匹配后，该实现被触发。因此，如果请求的扩展名是.aspx，便会生成Page类对象，而Page类对象是实现了IHttpHandler接口的。

2，在第九个到第十事件之间根据SessionId获取Session

第九到第十个事件是：`AcquireRequestState`，`PostAcquireRequestState`。

这期间首先会接收到浏览器发过来的SessionId，然后先会将IHttpHandler接口尝试转换为IRequiresSessionState接口，如果转换成功，ASP.NET会根据这个SessionId到服务器的Session池中去查找所对应的Session对象，并将这个Session对象赋值到HttpContext对象的Session属性。如果尝试转换为IRequiresSessionState接口不成功，则不加载Session。

3，在第十一个事件与第十二个事件之间执行页面生命周期

第十一和第十二个事件是：`PreRequestHandlerExecute`，`PostRequestHandlerExecute`。在这两个事件之间，ASP.NET最终通过请求资源类型相对应的HttpHandler实现对请求的处理，其实现方式是调用在第八个事件创建的页面对象的ProcessRequest方法。

![051837105625665.jpg](/img/051837105625665.jpg)

在 `FrameworkInitialize()` 这个方法内部就开始打造WebForm的页面控件树，在其中调用了ProcessRequestMain方法，在这个方法里面就执行了整个ASP.NET WebFom页面生命周期。

## MVC 管道事件摘要

在ASP.NET MVC中，最核心的当属“路由系统”，而路由系统的核心则源于一个强大的 `System.Web.Routing.dll` 组件。

<font color=##ff0000 size=4 face="黑体">在这个System.Web.Routing.dll中，有一个最重要的类叫做UrlRoutingModule , ASP.NET MVC的入口在 `UrlRoutingModule`，它是一个实现了`IHttpModule` 接口的类,订阅了 `HttpApplication` 的第7个管道事件 `PostResolveRequestCahce` 对请求进行了拦截。</font>

1,IIS网站的配置可以分为两个块：全局 Web.config 和本站 Web.config。Asp.Net Routing属于全局性的，所以它配置在全局Web.Config 中，我们可以在如下路径中找到：“$\Windows\Microsoft.NET\Framework\版本号\Config\Web.config“

```cs
<?xml version="1.0" encoding="utf-8"?>
 <!-- the root web configuration file -->
 <configuration>
     <system.web>
         <httpModules>
             <add name="UrlRoutingModule-4.0" type="System.Web.Routing.UrlRoutingModule" />
         </httpModules>
    </system.web>
 </configuration>
```

2,通过在全局Web.Config中注册 `System.Web.Routing.UrlRoutingModule`，IIS请求处理管道接到请求后，就会加载 UrlRoutingModule类型的Init()方法，第7个管道事件 `PostResolveRequestCahce` 对请求进行了拦截。

3,在第七个事件中创建实现了IHttpHandler接口的MvcHandler

第七个事件：`PostResolveRequestCache`

当请求到达 `UrlRoutingModule` 的时候，`UrlRoutingModule` 取出请求中的 Controller、Action等RouteData信息，与路由表中的所有规则进行匹配，若匹配，把请求交给 `IRouteHandler`，即 `MVCRouteHandler`。我们可以看下`UrlRoutingModule` 的源码来看看，以下是几句核心的代码：

```cs
public virtual void PostResolveRequestCache(HttpContextBase context)
{
    // 通过RouteCollection的静态方法GetRouteData获取到封装路由信息的RouteData实例
    RouteData routeData = this.RouteCollection.GetRouteData(context);
    if (routeData != null)
    {
        // 再从RouteData中获取MVCRouteHandler
        IRouteHandler routeHandler = routeData.RouteHandler;
        ......
        if (!(routeHandler is StopRoutingHandler))
        {
            ......
            // 调用 IRouteHandler.GetHttpHandler()，获取的IHttpHandler 类型实例，它是由 IRouteHandler.GetHttpHandler获取的
            IHttpHandler httpHandler = routeHandler.GetHttpHandler(requestContext);
            ......
            // 合适条件下，把之前将获取的IHttpHandler 类型实例 映射到IIS HTTP处理管道中
            context.RemapHandler(httpHandler);
        }
    }
}
```

`MVCRouteHandler` 的作用是用来生成实现 `IHttpHandler` 接口的 `MvcHandler`：

```cs
namespace System.Web.Routing
{  
    public interface IRouteHandler
    {
        IHttpHandler GetHttpHandler(RequestContext requestContext);
    }
}
```

获取 `MvcRouteHadnler` :

```cs
// 继承HttpApplication
public class MvcApplication : System.Web.HttpApplication
{
    // 第一个管道事件
    protected void Application_Start()
    {
        ......
        //注册路由
        RouteConfig.RegisterRoutes(RouteTable.Routes);
        BundleConfig.RegisterBundles(BundleTable.Bundles);
    }
}

public class RouteConfig
{
    public static void RegisterRoutes(RouteCollection routes)
    {
        routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
        // MapRoute位于System.Web.Mvc.RouteCollectionExtensions
        routes.MapRoute(
            name: "Default",
            url: "{controller}/{action}/{id}",
            defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
        );
    }
}

// System.Web.Mvc.RouteCollectionExtensions 扩展方法
public static Route MapRoute(this RouteCollection routes, string name, string url, object defaults, object constraints, string[] namespaces)
{
    ......
　　　// 通过Route类的构造函数把 MvcRouteHandler 注入到路由中
    Route route = new Route(url, new MvcRouteHandler())             {
        Defaults = new RouteValueDictionary(defaults),
        Constraints = new RouteValueDictionary(constraints),
        DataTokens = new RouteValueDictionary()
    };
    ......
    return route;
}
```

在第11和第12个事件之间调用了 `MvcHandler` 的 `ProcessRequest` 方法处理ASP.NET MVC。

第十一和第十二个事件是：`PreRequestHandlerExecute`，`PostRequestHandlerExecute`。

获取 `MvcHandler`:

```cs
IHttpHandler httpHandler = routeHandler.GetHttpHandler(requestContext);
context.RemapHandler(httpHandler);

// MvcHandlerd的实现
protected virtual IHttpHandler GetHttpHandler(RequestContext requestContext)
{
　　requestContext.HttpContext.SetSessionStateBehavior(GetSessionStateBehavior(requestContext));
    return new MvcHandler(requestContext);
}
```

如图：

![190104512548523.png](/img/190104512548523.png)

参考：

[再谈IIS与ASP.NET管道](https://www.cnblogs.com/artech/archive/2009/06/20/1507165.html)

[【深入ASP.NET原理系列】--ASP.NET请求管道、应用程序生命周期、整体运行机制](https://www.cnblogs.com/zhangyihui/p/5082394.html)

[ASP.NET MVC请求处理管道生命周期的19个关键环节(1-6)](http://www.cnblogs.com/darrenji/p/3795661.html)

[ASP.NET MVC请求处理管道生命周期的19个关键环节(7-12)](http://www.cnblogs.com/darrenji/p/3795676.html)

[ASP.Net请求处理机制初步探索之旅 - Part 3 管道](http://www.cnblogs.com/edisonchou/p/4201855.html)
