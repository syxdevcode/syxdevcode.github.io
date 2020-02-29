---
title: ASP.NET WebForm页面生命周期
date: 2019-02-18 11:32:37
tags:
- DotNet
- WebForm
- ASP.NET/IIS请求处理管道
categories: 
- WebForm
---
# ASP.NET WebForm页面生命周期

## 生命周期阶段

### 1、请求页面

页请求发生在页生命周期开始之前。

### 2、开始阶段

1，Page类的ProcessRequest方法作为页面生命周期的入口；
2，BuildControlTree：构造页面控件树；
3，设置IsPostBack属性是否第一次请求该页面；

### 3、初始化页面

页面初始化期间，可以使用页中的控件，并将设置每个控件的UniqueID属性。如果当前请求是回发请求，则回发数据尚未加载，并且控件属性值尚未还原为视图状态中的值。

事件：`PreInit-->Init-->InitComplete`

** PreInit**

PreInit 是页面生命周期的第一个事件。它检查 IsPostBack 属性并决定页面是否是回发。它设置主题及主版页，创建动态控件，并且获取和设置值配置文件属性值。此事件可通过重载 OnPreInit 方法或者创建一个 Page_PreInit 处理程序进行处置。

** Init**

Init 事件初始化控件属性，并且建立控件树。此事件可通过重载 OnInit 方法或者创建一个 Page_Init处理程序进行处置。

** InitComplete**

InitComplete 事件允许对视图状态的跟踪。所有的控件开启视图状态下的跟踪。

### 4、加载页面

设置控件属性，使用视图状态和控件状态值。

事件：

```cs
（LoadState-->ProcessPostData-->）PreLoad-->Load-->
（ProcessPostData-->RaiseChangedEvents-->RaisePostBackEvent-->）LoadComplete
```

** LoadViewState**

允许加载视图状态信息到控件中。

** LoadPostData**

对所有由 \ 标签定义的输入字段的内容进行处理。

** PreLoad**

PreLoad 在回发数据加载在控件中之前发生。此事件可以通过重载 OnPreLoad 方法或者创建一个 Pre_Load 处理程序进行处置。

** Load **

Load 事件被页面最先引发，然后递归地引发所有子控件。控件树中的控件就被创建好了。此事件可通过重载 OnLoad 方法或者创建一个 Page_Load 处理程序来进行处置。

### 5、验证

在验证期间，将调用所有验证程序控件的Validate方法，此方法将设置各个验证程序控件和页的IsValid属性为 true。

** LoadComplete**

加载进程完成，控件事件处理程序运行，页面验证发生。此事件可通过重载 OnLoad 方法或者创建一个 Page_LoadComplete 处理程序来进行处置。

### 6、回发事件处理

如果请求是回发请求，则将调用所有事件处理程序。

### 7、呈现页面

页面的视图状态和所有控件被保存。页面为每一个控件调用显示方法，并且呈现的输出被写入页面响应属性中的 OutputStream 类。

** PreRender**

PreRender 事件就在输出显示之前发生。通过处理此事件，页面和控件可以在输出显示之前执行任何更新。

** PreRenderComplete**

当 PreRender 事件为所有子控件被循环引发，此事件确保了显示前阶段的完成

** SaveStateComplete**

页面控件状态被保存。个性化、控件状态以及视图状态信息被保存。

### 8、卸载页面

显示过的页面被送达客户端以及页面属性，例如响应和请求，被卸载并全部清除完毕。

参考：

[用三张图片详解Asp.Net 全生命周期](http://www.cnblogs.com/zhaoyang/archive/2011/11/16/2251200.html)

[ASP.Net请求处理机制初步探索之旅 - Part 4 WebForm页面生命周期](http://www.cnblogs.com/edisonchou/p/4216337.html)

[ASP.NET 页面生命周期](https://www.w3cschool.cn/aspnet/1wiv1j2g.html)

[Asp.Net WebForm生命周期的详解](https://zhuanlan.zhihu.com/p/28318813)