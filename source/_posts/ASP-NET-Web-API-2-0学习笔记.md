---
title: ASP.NET Web API 2.0学习笔记
date: 2019-03-25 08:35:32
tags:
- DotNet
- ASP.NET/IIS请求处理管道
- MVC/Web API
categories: 
- MVC/Web API
---
# ASP.NET Web API 2.0学习笔记

## 概述

ASP.NET Web API是一个独立于传输层的抽象消息处理管道，它能根据请求采用 HTTP方法来确定目标 Action方法。
IIS 默认拒绝`PUT`和`DELETE`请求，原因是默认注册了 `WebDAVModule`，可以通过配置文件移除`HttpModule`。

** Web API的宿主方式**
1，web host 方式
    以 Web Host的 方式寄宿 Web API需 要做的唯——件事情是路 由注册。

2，self host 方式
    任意类型的应用程序 (控制台、Windows Forms应 用、WPF应用甚至是 Windows sewioc)作为宿主。
    除了必须的路 由注册外,我们还需要完成额外的一件事情,即手工加载定义了HttpController类型的程序集。

## 路由

### ASP.NET路由系统

一个Web 应用具有一个全局的路由表，通过 `System.Web.Routing.RouteTable`的静态属性`Routes`表示。

 RouteBase

一个`Route`继承抽象类`System.Web.Routing.RouteBase`。

  RouteData

VirTalPathData

Route
`System.Web.Routing.Route`是抽象类`System.Web.Routing.RouteBase`唯一的直接子类，基于路由模版模式的路由匹配规则就定义在`Route`中。

RouteTable

### ASP.NET WEB API 路由

请求与响应

ASP.NET WEB API通过类型`HttpRequestMessage`和`HttpResponseMessage`表示的管道处理的请求和响应的消息，定义在`System.Net.Http.dll`中。

HttpRequestMessage

HttpResponseMessage

HttpContent

#### ASP.NET WEB API 路由系统

ASP.NET WEB API路由系统中`HostedHttpRoute`对象通过创建ASP.NET路由系统中的`HttpWebRoute`对象进行路由解析。
针对约束的检查则依然是ASP.NET WEB API路由系统中的`HttpRouteConstraint`。

Web Host宿主模式采用的route类型为`HttpWebRoute`,对应的`RouteHandler`是`HttpControllerRouteHandler`对象，提供的`HttpHandler`类型为`HttpControllerHandler`。

ASP.NET WEB API以Web Host模式部署并注册相应的路由后，这些注册的`HttpRoute(HostedHttpRoute)`最终会转换成asp.net全局路由表中的`Route(HttpWebRoute)`
asp.net路由系统对请求进行拦截，如果匹配某个Route,响应的路由数据
被解析出来并保存在`RequestContext`中，随后asp.net路由系统的实现者
`UrlRoutingModule`从匹配的Route中获取`RouteHandler`,即`HttpControllerRouteHandler`对象，该对象提供`HttpHandler(HttpControllerHandler)`被映射到当前请求。
一旦映射成功，`HttpControllerHandler`将最终接管当前请求，它会构建一个
消息处理管道来处理这个请求并对请求予以响应。

`HttpControllerHandler`创建`HttpRequestMessage`，并将ASP.NET路由系统解析得到的`RouteData`转换成`HttpRouteData,`并添加到`HttpRequestMessage`属性的字典中。

HttpRouteData

HttpVirtualPathData

HttpRouteConstraint

HttpRoute

HttpRouteCollection

HostedHttpRoute
: 可以看成一个对Route对象的封装。

## 消息处理管道

ASP.NET WEB API的核心框架是一个消息处理管道，由一组`HttpMessageHandler`的有序组合。

1, HttpMessageHandler
ASP.NET WEB API的核心框架是一个消息处理管道,由一组`HttpMessageHandler`经过收尾相连而成。

2，DelegatingHandler

管道的串联通过`DelegatingHandler`来完成。

3，HttpServer

作为`HttpMessageHandler`链的龙头

4，HttpRoutingDispatcher

消息处理管道最后一个`HttpMessageHandler`.
主要功能：路由(Routing)和消息分发（Dispatching）
默认从`HttpRoutingDispatcher`接管请求的`HttpMessageHandler`是`HttpControllerDispatcher`对象。对`HttpController`的激活和Action方法的执行和响应都是有`HttpControllerDispatcher`完成的。

### Web Host模式下消息处理管道

HttpControllerHandler

`HttpControllerHandler`创建`HttpRequestMessage`，并将ASP.NET路由系统解析得到的`RouteData`转换成`HttpRouteData,`并添加到`HttpRequestMessage`属性的字典中。

### Self Host模式下消息处理管道

## HttpController的激活

### HttpController

`HttpRoutingDispatcher`会利用隶属于它的`HttpControllerDispatcher`激活目标`HttpController`对象。

1，HttpControllerContext

`HttpController`对象`ExecuteAsync`方法得到`HttpControllerContext`。

2，HttpControllerDescripor

`HttpControllerDescriptor`封装了某个`HttpController`类型的元数据，即`HttpController`类型的描述对象。

创建`HttpController`

### HttpController如何创建的

`HttpControllerDispatcher`实现目标`HttpController`对象的激活与执行，并将代表执行结果的`HttpResponseMessage`对象返回给`HttpRoutingDispatcher`对象,后者将`HttpResponseMessage`回传给消息管道进行相应的处理后最终完成对请求的响应。

`HttpControllerDispatcher`接管请求后，它会获取注册的`HttpControllerSelector`对象,默认注册的`HttpControllerDescriptor` 是一个`DefaultHttpControllerSelector`,后者借助于注册的`HttpControllerTypeResolver`对象得到所有的`HttpController`类型，进而创建一个描述这些`HttpController`的`HttpControllerDescriptor`对象与`HttpController`名称之间的映射关系，并调用其`SelectController`方法 得到描述目标`HttpController`的`HttpControllerDescriptor`对象。

`HttpControllerDispatcher`接下来调用这个`HttpControllerDescriptor`对象的`CreateController`方法得到激活的`HttpController`对象。对于这个`HttpControllerDescriptor`对象来说，当它`CreateController`方法被调用之后，它会获取注册的`HttpControllerActivator`对象，并调用其Create方法实现针对目标`HttpController`对象的激活并将激活的对象返回。

默认注册的`DefaultHttpControllerActivator`对象会利用注册的`DependencyResolver`根据`HttpController`类型去获取代表目标`HttpController`实例的对象。如果后者返回一个`HttpController`对象，该对象将直接作为方法的返回值，否则`DefaultHttpControllerActivator`直接采用反射的形式创建目标`HttpController`对象并返回。

由于默认注册的`DepandencyResolver`是一个`EmptyResolver`对象，由它返回`HttpController`对象总是null,所以在默认情况下激活`HttpController`对象总是以反射的形式创建的。所以我们定义`HttpController`类型必须具有一个默认的构造函数。

## Action的选择

### HttpActionDescriptor

`HttpActionDescriptor`是一个抽象类。

定义在`HttpController`类型中的每一个Action方法都是通过`HttpActionDescriptor`对象来描述的。描述Action方法的基本元数据信息均可以在对应的`HttpActionDescriptor`对象中找到。

### ReflectedHttpActionDescriptor

`HttpActionDescriptor`是一个抽象类,默认实现是`ReflectedHttpActionDescriptor`对象，通过对目标方法实施反射来获取相关的元数据信息。
一个`ReflectedHttpActionDescriptor`对象是对一个`MethodInfo`的封装,
描述Action方法相关的元数据基本来源于此。

### ActionNameAttribute

应用`ActionNameAttribute`特性来对Action进行命名。

### 方法名决定HTTP方法

如果Action方法具有以下的前缀（不区分大小写），那么对应的HTTP方法将默认支持。不具有，则默认支持POST。

* GET
* POST
* PUT
* DELETE
* HEAD
* OPTIONS
* PATCH

### ActionHttpMethodProvider

`ActionHttpMethodProvider`应用到某个Action方法上，并为其提供它所支持的HTTP方法。

AcceptVerbsAttribute特性

写法：

```cs
[AcceptVerbsAttribute("PUT","POST")]
```

ASP.NET Web API 定义的7中对应的特性类型：

```cs
HttpGetAttribute
HttpPostAttribute
HttpPutAttribute
HttpDeleteAttribute
HttpHeadAttribute
HttpOptionsAttribute
HttpPatchAttribute
```

### HttpParameterDescriptor

抽象类，用户描述Action方法参数的`HttpParameterDescriptor`对象。

### ReflectedHttpParameterDescriptor

### HttpActionSelector

使用`ApiControllerActionSelector`对象作为默认的`HttpActionSelector`。

## 特性路由




参考：

[ASP.NET Web API 2 框架揭秘](ASP.NET Web API 2 框架揭秘)