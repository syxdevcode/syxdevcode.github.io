---
title: ASP.NET在IIS运行模式
date: 2019-02-15 08:55:33
tags:
- ASP.NET/IIS请求处理管道
- DotNet
categories: 
- ASP.NET/IIS请求处理管道
---
# ASP.NET在IIS运行模式

## IIS6 处理模式

&emsp;&emsp;IIS 6引入了Application Pool。Application Pool就是一个application的容器，在IIS 6中，我们可以创建若干Application Pool，在创建Web Application的时候，我们为它指定一个既定的application pool。在运行的时候，一个Application对应一个Worker Process：w3wp.exe。也就是说，和前一个版本的IIS不同的是，对于IIS 6来说，同一台机器上可以同时运行多个Worker Process，每个Worker Process中的每个Application domain对应一个Application。这样，Application之间不但能提供Application Domain级别的隔离，你也可以将不同的Application置于不同的Application Pool中，从而基于Process级别的隔离。对于Host 一些重要的Application来说，这样的方式可以提供很好的Reliability。

注：<font color=##ff0000 size=4 face="黑体">为了避免用户应用程序访问或者修改关键的操作系统数据，windows提供了两种处理器访问模式：用户模式（User Mode）和内核模式（Kernel Mode）。一般地，用户程序运行在User mode下，而操作系统代码运行在Kernel Mode下。Kernel Mode的代码允许访问所有系统内存和所有CPU指令。</font>

&emsp;&emsp;在User Mode下，http.sys接收到一个基于aspx的http request，然后它会根据IIS中的Metabase查看该基于该Request的Application属于哪个Application Pool，如果该Application Pool不存在，则创建之。否则直接将request发到对应Application Pool的Queue中。每个Application Pool对应着一个Worker Process：w3wp.exe，是运行在User Mode下的。在IIS Metabase中维护着Application Pool和worker process的Mapping。WAS（Web Administrative service）根据这样一个mapping，将存在于某个Application Pool Queue的request 传递到对应的worker process(如果没有，就创建这样一个进程)。在worker process初始化的时候，加载ASP.NET ISAPI，ASP.NET ISAPI进而加载CLR。最后的流程就和IIS 5.x一样了：通过AppManagerAppDomainFactory的Create方法为Application创建一个Application Domain；通过ISAPIRuntime的ProcessRequest处理Request，进而将流程进入到ASP.NET Http Runtime Pipeline。

![IIS6-1.jpg](/img/IIS6-1.jpg)

* HTTP.SYS:（Kernel）的一个组件，它负责侦听（Listen）来自于外部的HTTP请求,根据请求的URL将其转发给相应的应用程序池 (Application Pool)。当此HTTP请求处理完成时，它又负责将处理结果发送出去.为了提供更好的性能，HTTP.SYS内部建立了一个缓冲区，将最近的HTTP请求处理结果保存起来。
* Application Pool:IIS总会保持一个单独的工作进程：应用程序池。所有的处理都发生在这个进程里，包括ISAPI dll的执行。对于IIS6而言，应用程序池是一个重大的改进，因为它们允许以更小的粒度控制一个指定进程的执行。你可以为每一个虚拟目录或者整个Web 站点配置应用程序池，这可以使你很容易的把每一个应用程序隔离到各自的进程里，这样就可以把它与运行在同一台机器上其他程序完全隔离。从Web处理的角度看，如果一个进程死掉，至少它不会影响到其它的进程。当应用程序池接收到HTTP请求后，交由在此应用程序池中运行的工作者进程Worker Process: w3wp.exe来处理此HTTP请求。 
* Worker Process: 当工作者进程接收到请求后，首先根据后缀找到并加载对应的ISAPI扩展 (如:aspx 对应的映射是aspnet_isapi.dll)，工作者进程加载完aspnet_isapi.dll后，由aspnet_isapi.dll负责加载 ASP.NET应用程序的运行环境即CLR (.NET Runtime)。 Worker Process运行在非托管环境，而.NET中的对象则运行在托管环境之上(CLR)，它们之间的桥梁就是ISAPI扩展。
* WAS（Web Admin Service）：这是一个监控程序，它一方面可以存取放在InetInfo元数据库（Metabase）中的各种信息，另一方面也负责监控应用程序池（Application Pool）中的工作者进程的工作状态况，必要时它会关闭一个老的工作者进程并创建一个新的取而代之。

### IIS6.X 运行时

在worker process(w3wp.ext)初始化的时候，加载ASP.NET ISAPI，ASP.NET ISAPI进而加载CLR。
<font color=##ff0000 size=4 face="黑体">ASP.NET ISAPI运行在一个非托管环境之中,ASP.NET ISAPI通过调用System.Web.Hosting.ISAPIRuntime Instance的ProcessRequest方法，进而从非托管的环境进入了托管的环境。</font>

![iis_aspnet_process_mode_02_01.JPG](/img/iis_aspnet_process_mode_02_01.JPG)

具体请参考：[ASP.NET Process Model之二：ASP.NET Http Runtime Pipeline[上篇]](http://www.cnblogs.com/artech/archive/2007/09/13/891262.html)

最后，通过 `AppManagerAppDomainFactory` 的Create方法为 `Application` 创建一个 `Application Domain`；通过`ISAPIRuntime` 的 `ProcessRequest` 处理 `Request`，进而将流程进入到 `ASP.NET Http Runtime Pipeline`。

如下图：

![2012071209470854.png](/img/2012071209470854.png)

<font color=##ff0000 size=4 face="黑体">HttpContext(Request,Response)→ HttpRuntime→HttpApplicationFactory→HttpApplication→ HttpModule→HttpHandler→EndRequest</font>

![iis_aspnet_process_mode_02_02.JPG](/img/iis_aspnet_process_mode_02_02.JPG)

## IIS7 处理模式

IIS 7.0支持经典模式和集成模式。IIS6及IIS7的经典模式需要aspnet_isapi.dll来处理，而IIS7集成模式不需要aspnet_isapi.dll来处理，而可以直接根据文件扩展名找到相应的处理程序接口。例如aspx的处理程序是System.Web.UI.PageHandlerFactory类型。

IIS 7.0对请求的监听和分发机制上又进行了革新性的改进，主要体现在对于Windows进程激活服务（Windows Process Activation Service，WAS）的引入，将原来（IIS 6.0）W3SVC承载的部分功能分流给了WAS。具体来说，通过上面的介绍，我们知道对于IIS 6.0来说，W3SVC主要承载着三大功能：

* HTTP请求接收：接收HTTP.SYS监听到的HTTP请求；
* 配置管理：从元数据库（Metabase）中加载配置信息对相关组件进行配置；
* 进程管理：创建、回收、监控工作进程。

在IIS 7.0，后两组功能被移入WAS中，接收HTTP请求的任务依然落在W3SVC头上。

### 经典模式

经典模式与IIS 6或者之前版本保持兼容的一种模式。

利用文件扩展名，可以判断使用哪个ISAPI处理程序。例如，可以将扩展名为.aspx 和.ascx的文件映射到aspnet_isapi.dll；并且将扩展名为.asp的文件映射到asp.dll，这样就可以处理传统的ASP页面。

![04103814-376a25a1abe84619a45a90e3882ac641.jpg](/img/04103814-376a25a1abe84619a45a90e3882ac641.jpg)

### 集成模式

![04103815-d7ef2284320f485d9f048b507062cc4d.png](/img/04103815-d7ef2284320f485d9f048b507062cc4d.png)

1、当客户端浏览器开始 HTTP 请求一个WEB 服务器的资源时，HTTP.sys 拦截到这个请求。

2、HTTP.sys 联系 WAS 获取配置信息。

3、WAS 向配置存储中心(applicationHost.config)请求配置信息。

4、WWW 服务接收到配置信息，配置信息指类似应用程序池配置信息，站点配置信息等等。

5、WWW 服务使用配置信息去配置 HTTP.sys 处理策略。

6、WAS为请求创建一个进程(如果不存在的话)

7、工作者进程处理请求并对HTTP.sys做出响应.

8、客户端接受到处理结果信息。

&emsp;&emsp;集成模式中，asp.net不再像IIS6一样只限定于aspnet_isapi.dll中，而是被解放出来，从IIS接收到HTTP请求开始，即进入asp.net的控制范围，asp.net可以存在于一个请求在IIS中各个处理阶段。允许我们将ASP.NET更好地与IIS集成，甚至允许我们在ASP.NET中编写一些功能（例如Module）来改变IIS的行为（扩 展）。集成的好处是，不再通过ISAPI的方式，提高了速度和稳定性。至于扩展，则可以使得我们对于IIS，以及其他类型的请求有更多的控制。（例如，我们希望静态网页也具备一些特殊的行为）。

IIS 7.0集成模式下的优点：

* 允许我们通过本地代码（Native Code）和托管代码（Managed Code）两种方式定义IIS Module，这些IIS Module注册到IIS中形成一个通用的请求处理管道。由这些IIS Module组成的这个管道能够处理所有的请求，不论请求基于怎样的资源类型。比如，可以将FormsAuthenticationModule提供的Forms认证应用到基于.aspx，CGI和静态文件的请求。
* 将ASP.NET提供的一些强大的功能应用到原来难以企及的地方，比如将ASP.NET的URL重写功能置于身份验证之前；
* 采用相同的方式去实现、配置、检测和支持一些服务器特性（Feature），比如Module、Handler映射、错误定制配置（Custom Error Configuration）等。

参考：

[ASP.NET Process Model之一：IIS 和 ASP.NET ISAPI](http://www.cnblogs.com/artech/archive/2007/09/09/887528.html)

[用三张图片详解Asp.Net 全生命周期](http://www.cnblogs.com/zhaoyang/archive/2011/11/16/2251200.html)

[IIS与ASP.NET Http Runtime Pipeline](http://www.cnblogs.com/tenghoo/archive/2009/11/04/IIS_And_ASPNET_Http_Runtime_Pipeline.html)

[瀚海拾贝（一）HTTP协议/IIS 原理及ASP.NET运行机制浅析【图解】](http://www.cnblogs.com/wenthink/archive/2013/05/06/HTTP_IIS_ASPNET_Pipeline.html)

[ASP.NET是如何在IIS下工作的](http://www.cnblogs.com/fengzheng/p/3668283.html)