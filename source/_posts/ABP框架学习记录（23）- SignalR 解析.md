---
title: ABP框架学习记录（23）- SignalR 解析
date: 2019-09-21 09:47:55
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（23）- SignalR解析

SignalR是一个.NET Core/.NET Framework的开源实时框架. SignalR的可使用Web Socket, Server Sent Events 和 Long Polling作为底层传输方式.

SignalR基于这三种技术构建, 抽象于它们之上, 它让你更好的关注业务问题而不是底层传输技术问题.

SignalR这个框架分服务器端和客户端, 服务器端支持ASP.NET Core 和 ASP.NET; 而客户端除了支持浏览器里的javascript以外, 也支持其它类型的客户端, 例如桌面应用.

整体目录：

![QQ截图20190921105001.png](/img/QQ截图20190921105001.png)

`AbpHubBase`： 继承 `Hub` 抽象类，定义成 Abp SignalR 的抽象类，类成员包括 `AbpSession`,`IocResolver`,`LocalizationManager` 等；

![QQ截图20190921110241.png](/img/QQ截图20190921110241.png)

`AbpHubBaseOfClientType`：定义泛型 `Hub`;

![QQ截图20190921110421.png](/img/QQ截图20190921110421.png)

`OnlineClientHubBase`：在线客户端 `Hub` 基类；

![QQ截图20190921110621.png](/img/QQ截图20190921110621.png)

![QQ截图20190921110755.png](/img/QQ截图20190921110755.png)

`AbpCommonHub`：公用 `Hub`,提供注册方法；

![QQ截图20190921111019.png](/img/QQ截图20190921111019.png)

`SignalRRealTimeNotifier`： 实现 `IRealTimeNotifier` 接口；

![QQ截图20190921114233.png](/img/QQ截图20190921114233.png)

![QQ截图20190921134226.png](/img/QQ截图20190921134226.png)

`AbpAspNetCoreSignalRModule`：定义 `SignalRModule`;

![QQ截图20190921134356.png](/img/QQ截图20190921134356.png)

参考：

[ASP.NET Core的实时库: SignalR -- 预备知识](https://www.cnblogs.com/cgzl/p/9509207.html)

[ASP.NET Core的实时库: SignalR简介及使用](https://www.cnblogs.com/cgzl/p/9515516.html)