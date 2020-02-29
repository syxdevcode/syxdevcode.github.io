---
title: ABP框架学习记录（22）- RealTime的实现
date: 2019-09-12 09:44:13
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（22）-RealTime的实现

`RealTime`，翻译为 即时的，实时的；

`RealTime` 的实现，在ABP项目中的位置：

![QQ截图20190912094652.png](/img/QQ截图20190912094652.png)

`IOnlineClientStore`：提供存储接口；

![QQ截图20190920151501.png](/img/QQ截图20190920151501.png)

`InMemoryOnlineClientStore`：内存存储的实现；

![QQ截图20190920151559.png](/img/QQ截图20190920151559.png)

`IOnlineClient`：表示一个连接到应用的客户端；

![QQ截图20190920151741.png](/img/QQ截图20190920151741.png)

`OnlineClient`：`IOnlineClient` 接口的实现；

![QQ截图20190920152302.png](/img/QQ截图20190920152302.png)

`OnlineClientEventArgs`：客户端事件

![QQ截图20190920152812.png](/img/QQ截图20190920152812.png)

`OnlineUserEventArgs`：用户事件

![QQ截图20190920153024.png](/img/QQ截图20190920153024.png)

`IOnlineClientManager`：管理连接到应用的客户端；

![QQ截图20190920155746.png](/img/QQ截图20190920155746.png)

`OnlineClientManager`：`IOnlineClientManager` 的实现；

![QQ截图20190920160558.png](/img/QQ截图20190920160558.png)

![QQ截图20190920160908.png](/img/QQ截图20190920160908.png)