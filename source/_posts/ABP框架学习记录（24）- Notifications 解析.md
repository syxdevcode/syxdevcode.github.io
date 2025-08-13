---
title: ABP框架学习记录（24）- Notifications 解析
date: 2019-09-23 09:39:48
tags:
- DotNet
- CSharp
- ABP
categories: 
- ABP
---
# ABP框架学习记录（24）- Notifications 解析

`Notifications` 封装了通知的功能，实现 `Notification` 的定义，存储，发送，订阅等功能，支持租户通知，用户通知。通过 `IRealTimeNotifier` 接口，实现统一的 `Notification` 功能。支持 `SignalR`。

`Notifications` 在ABP项目中的位置：
![QQ截图20190923141119.png](/img/QQ截图20190923141119.png)

## 表-实体类对应

首先，介绍关于 `Notifications` 的表-实体类；

`NotificationInfo`：用于存储需要发送的通知；

![QQ截图20190923181330.png](/img/QQ截图20190923181330.png)

`NotificationSubscriptionInfo`: 用于存储 通知订阅。

![QQ截图20190923181457.png](/img/QQ截图20190923181457.png)

`TenantNotificationInfo`: 分配给其相关租户的通知。

![QQ截图20190924092140.png](/img/QQ截图20190924092140.png)

`UserNotificationInfo`：用户存储用户通知

![QQ截图20190924092240.png](/img/QQ截图20190924092240.png)

## 通知数据

`NotificationData`：用户 通知数据的 存储；

![QQ截图20190924134512.png](/img/QQ截图20190924134512.png)

`MessageNotificationData`：继承 `NotificationData`, 可以用户简单的通知消息；

![QQ截图20190924134619.png](/img/QQ截图20190924134619.png)

`LocalizableMessageNotificationData`：继承 `NotificationData`, 可以用户表示本地的通知消息数据；

![QQ截图20190924134652.png](/img/QQ截图20190924134652.png)

## 通知定义管理

`NotificationDefinition`：定义通知

![QQ截图20190924143636.png](/img/QQ截图20190924143636.png)

`NotificationDefinitionManager`：实现 `INotificationDefinitionManager` 接口，用于管理 `NotificationDefinition`;

![QQ截图20190924143805.png](/img/QQ截图20190924143805.png)

`AbpKernelModule`：`PostInitialize` 方法中初始化；

![QQ截图20190924144304.png](/img/QQ截图20190924144304.png)

`NotificationDefinitionContext`：定义 `NotificationDefinition` 上下文；

![QQ截图20190924143854.png](/img/QQ截图20190924143854.png)

`NotificationDefinitionManagerExtensions`：扩展方法，获取给定用户的所有通知；

![QQ截图20190924144047.png](/img/QQ截图20190924144047.png)

## 配置

`INotificationConfiguration`：配置通知；

`NotificationConfiguration`：实现 `INotificationConfiguration` 接口；

![QQ截图20190924150138.png](/img/QQ截图20190924150138.png)

`AbpCoreInstaller` 中注册：

![QQ截图20190924150305.png](/img/QQ截图20190924150305.png)

## 持久化

`INotificationStore`：定义通知持久化接口；

![QQ截图20190924151312.png](/img/QQ截图20190924151312.png)

`Abp.Zero.Common` 项目中，使用仓储模式，提供 `INotificationStore` 接口的实现 `NotificationStore`；

![QQ截图20190924151629.png](/img/QQ截图20190924151629.png)

`UserNotificationState`：定义通知的状态，已读，未读；

## 管理用户通知

`IUserNotificationManager`：定义用户通知管理接口；

`UserNotificationManager`：实现 `IUserNotificationManager` 接口；

![QQ截图20190924165743.png](/img/QQ截图20190924165743.png)

`UserNotification`：表示发送给用户的通知；

![QQ截图20190924165934.png](/img/QQ截图20190924165934.png)

`TenantNotification`：代表租户/用户 已经发送的通知；

![QQ截图20190924170219.png](/img/QQ截图20190924170219.png)

## 发布通知

`INotificationPublisher`：定义发布通知的接口；

`NotificationPublisher`：实现 `INotificationPublisher` 接口；

![QQ截图20190924170626.png](/img/QQ截图20190924170626.png)

`PublishAsync` 异步方法:

![QQ截图20190924170842.png](/img/QQ截图20190924170842.png)

`NotificationDistributionJob`：后台工作，去通知用户；

![QQ截图20190924172001.png](/img/QQ截图20190924172001.png)

## 订阅通知

`NotificationSubscription`：表示一个用户订阅的通知；

`INotificationSubscriptionManager`：管理订阅通知；

`NotificationSubscriptionManager`：实现 `INotificationSubscriptionManager`；

![QQ截图20190924171659.png](/img/QQ截图20190924171659.png)

## 通知设置

`NotificationSettingNames`：定义通知开关 配置名；

![QQ截图20190924172434.png](/img/QQ截图20190924172434.png)

`NotificationSettingProvider`：适配器

![QQ截图20190924172654.png](/img/QQ截图20190924172654.png)

`AbpKernelModule` 模块初始化：

![QQ截图20190924172751.png](/img/QQ截图20190924172751.png)

## 发送通知

`INotificationDistributer`：定义发送给用户通知的接口；

`DefaultNotificationDistributer`：实现 `INotificationDistributer` 接口；

![QQ截图20190924173343.png](/img/QQ截图20190924173343.png)

![QQ截图20190924173922.png](/img/QQ截图20190924173922.png)

![QQ截图20190924174210.png](/img/QQ截图20190924174210.png)

`IRealTimeNotifier`：定义向用户发送实时通知的接口；