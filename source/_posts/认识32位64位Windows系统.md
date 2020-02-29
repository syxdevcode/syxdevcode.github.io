---
title: 认识32位64位Windows系统
date: 2019-12-25 18:15:04
tags:
- Windows系统
- 计算机基础
- 程序集
categories: 
- Windows系统
---
## Wow64

Wow64，全称是32bit Windows On 64bit Windows（64位Windows上的32位Windows）。

Windows系统的主要系统文件都是放在一个叫做System32的文件夹中的。为了能同时放下两套系统文件，Windows会在64位的系统上，增加了一个文件夹，叫SysWow64。

总结：

```cs
SysWow64文件夹，是64位Windows，用来存放32位Windows系统文件的地方，而System32文件夹，是用来存放64位程序文件的地方。
当32位程序加载System32文件夹中的dll时，操作系统会自动映射到SysWow64文件夹中的对应的文件。
只要32位程序访问System32文件夹，无论是加载dll，还是读取文本信息，都会被映射到SysWow64文件夹
```

参考：

[什么是SysWow64](https://blogs.msdn.microsoft.com/tianlin/2011/10/26/syswow64/)

[dll文件32位64位检测工具以及Windows文件夹SysWow64的坑](https://www.cnblogs.com/hbccdf/p/dllchecktoolandsyswow64.html)

[【结果很简单,过程很艰辛】记阿里云Ons消息队列服务.NET接口填坑过程](https://www.cnblogs.com/asxinyu/p/dotnet_api_alibaba_ons_service_MQ.html)

[玩转Windows服务系列——Debug、Release版本的注册和卸载，及其原理](https://www.cnblogs.com/hbccdf/p/3486565.html)

[Windows Sysinternals](https://docs.microsoft.com/zh-cn/sysinternals/)

[live.sysinternals.com](https://live.sysinternals.com/)