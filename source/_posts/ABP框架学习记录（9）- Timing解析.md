---
title: ABP框架学习记录（9）- Timing解析
date: 2019-05-09 15:06:04
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（9）- Timing解析

Timing这个功能主要用于以统一的方式表示时间。因为ABP中有大量的module,还支持自定义module，所以将时间统一表示为local时间（默认）或utc时间是必要的。

## Clock（时钟）

`TimingSettingProvider`：继承 `SettingProvider` 以设置统一的时间格式。

![QQ截图20190509150917.png](/img/QQ截图20190509150917.png)

在 `AbpKernelModule` 的 `PreInitialize` 方法中引用：

![QQ截图20190509151217.png](/img/QQ截图20190509151217.png)

![QQ截图20190509151246.png](/img/QQ截图20190509151246.png)


ABP 提供 `IClockProvider` 获取当前时间和标准化时间的接口，三个实现 `IClockProvider`接口的类： `UtcClockProvider`，`UnspecifiedClockProvider`，`LocalClockProvider`。

`IClockProvider`：

![QQ截图20190509151848.png](/img/QQ截图20190509151848.png)

`LocalClockProvider`：

![QQ截图20190509152004.png](/img/QQ截图20190509152004.png)

`UtcClockProvider`：

![QQ截图20190509152201.png](/img/QQ截图20190509152201.png)

`UnspecifiedClockProvider`：

![QQ截图20190509152238.png](/img/QQ截图20190509152238.png)

`ClockProviders`：提供三种 `Providers`。

![QQ截图20190509152730.png](/img/QQ截图20190509152730.png)

`Clock`：封装了 `IClockProvider`，对外提供当前时间和标准化时间的方法。默认使用 `UnspecifiedClockProvider`。

![QQ截图20190509152858.png](/img/QQ截图20190509152858.png)

也可以指定 `Provider`：

```cs
Clock.Provider = ClockProviders.Utc;
```

## DateTimeRange（时间区间）

`IDateTimeRange/DateTimeRange`：表示一个时间区间的实体。

![QQ截图20190509155915.png](/img/QQ截图20190509155915.png)

使用：

![QQ截图20190509160338.png](/img/QQ截图20190509160338.png)

`IZonedDateTimeRange/ZonedDateTimeRange`：使用时区定义 `DateTime` 的范围。

![QQ截图20190509155958.png](/img/QQ截图20190509155958.png)

`TimeZoneConverter/ITimeZoneConverter`：时区转换类。

![QQ截图20190509164413.png](/img/QQ截图20190509164413.png)

`TimezoneHelper`：用于时区操作的帮助程序类。

![QQ截图20190509164913.png](/img/QQ截图20190509164913.png)

`IanaTimeZone` ：时区信息数据库，又称TZ database、Zoneinfo database，是一个主要应用于计算机程序以及操作系统的，可协作编辑世界时区信息的数据库。由于该数据库由David Olson创立，因而有些地方也将其称作Olson数据库。数据库由Paul Eggert进行编辑和维护

参考：

[ABP源码分析十一：Timing](http://www.cnblogs.com/1zhk/p/5317003.html)

[时区信息数据库/IANA time zone](https://zh.wikipedia.org/wiki/%E6%97%B6%E5%8C%BA%E4%BF%A1%E6%81%AF%E6%95%B0%E6%8D%AE%E5%BA%93)