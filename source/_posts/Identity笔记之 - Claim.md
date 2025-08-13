---
title: Identity笔记之 - Claim
date: 2019-10-17 14:45:07
tags:
- DotNet
- Identity
categories: 
- 认证与授权
---
# Identity笔记之 - Claim

本文介绍 `Claim` 相关概念，具体类在 `System.Security.Claims` 命名空间下。

## Claim

`Claim`：要求；主张；声称; 宣称; 自称; 认领

在 `Identity` 可以理解成 `证件单元`，以键值对表示：

例如：`姓名：特斯拉`

![QQ截图20191017150008.png](/img/QQ截图20191017150008.png)

## ClaimsIdentity

![QQ截图20191017150602.png](/img/QQ截图20191017150602.png)

在 `Identity` 可以理解成 `身份证件`，即包括一系列 `Claim` 的集合；

## ClaimsPrincipal

`Principal`：主体，委托人

`ClaimsPrincipal` 可以理解成 `证件当事人`;

![QQ截图20191017151234.png](/img/QQ截图20191017151234.png)

参考：

[ASP.NET Core 之 Identity 入门（一）](https://www.cnblogs.com/savorboard/p/aspnetcore-identity.html)