---
title: OpenID Connect 协议
date: 2019-03-19 10:52:32
tags:
- OAuth2.0
- OIDC
categories: 
- OAuth2.0
---
# OpenID Connect 协议

## OpenID Connect简介

`OpenID Connect`是由OpenID基金会于2014年发布的一个开放标准，简称OIDC, 它使用OAuth2.0来进行身份认证,`OpenID Connect`是OpenID的第三代技术。 `OpenID Connect`直接构建于OAuth2.0的基础之上,添加了一些组件来提供身份认证的能力与其兼容。 通常`OpenID Connect`是和OAuth2一同部署来使用的。

<font color=#ff0000 size=4 face="黑体">OpenID Connect是更高级的协议, 它扩展并替代了OAuth2.0 尽管现在我们经常说我们在使用OAuth2.0来保护API, 其实更准确的说, 大多数情况下, 我们使用的是OpenID Connect.</font>

身份认证(Authentication)
授权(Authorization)

## 名词定义

* EU：End User，用户。

* RP：Relying Party ，用来代指OAuth2中的受信任的客户端，身份认证和授权信息的消费方；

* OP：OpenID Provider，有能力提供EU身份认证的服务方（比如OAuth2中的授权服务），用来为RP提供EU的身份认证信息；

* ID-Token：JWT格式的数据，包含EU身份认证的信息。

* UserInfo Endpoint：用户信息接口（受OAuth2保护），当RP使用ID-Token访问时，返回授权用户的信息，此接口必须使用HTTPS。

## OIDC基础

OpenID是 `Authentication`，即认证，对用户的身份进行认证，判断其身份是否有效，也就是让网站知道“你是你所声称的那个用户”；

OAuth是 `Authorization`，即授权，在已知用户身份合法的情况下，经用户授权来允许某些操作，也就是让网站知道“你能被允许做那些事情”。

由此可知，授权要在认证之后进行，只有确定用户身份才能授权。
<font color=#ff0000 size=4 face="黑体">(身份验证)+ OAuth 2.0 = OpenID Connect</font>

`OpenID Connect`是“认证”和“授权”的结合，因为其基于OAuth协议，所以`OpenID Connect`协议中也包含了`client_id`、`client_secret`还有`redirect_uri`等字段标识。这些信息被保存在`身份认证服务器`，以确保特定的客户端收到的信息只来自于合法的应用平台。这样做是目的是为了防止`client_id`泄露而造成的恶意网站发起的OIDC流程。

在OAuth中的`scope`，`OpenID Connect`也有自己特殊的`scope--openid` ,它必须在第一次请求“身份鉴别服务器”（Identity Provider,简称IDP）时发送过去。

## OIDC流程

OIDC基于OAuth2，所以OIDC的认证流程主要是由OAuth2的几种授权流程延伸而来的，有以下3种：

* Authorization Code Flow：使用OAuth2的授权码来换取`Id_Token`和`Access_Token`。
* Implicit Flow：使用OAuth2的Implicit流程获取`Id_Toke`n和`Access_Token`。
* Hybrid Flow：混合`Authorization Code Flow`+`Implici Flow`。

OpenID Connect抽象流程：

![986268-20180627101906559-326862209.png](/img/986268-20180627101906559-326862209.png)

```txt
1. 依赖发(RP)发送请求到OpenID提供商(OP, 也就是身份提供商).
2. OpenID提供商验证最终用户的身份, 并获得了用户委派的授权
3. OpenID提供商返回响应, 里面带着ID_Token, 也通常带着Access_Token.
4. 依赖方现在可以使用Access_Token发送请求到用户信息的端点.
5. 用户信息端点返回用户的声明(claims, 相当于是用户的信息).
```

OpenID Connect身份认证有三个路径(三个流程, flow): `Authorization Code`, `Implicit`，`Hybrid`。

### Authorization Code(授权码)

要求客户端应用可以安全的在它和授权服务器之间维护客户端的`secret`,也就是说只适合这样的客户端应用。它还适合于长时间的访问(通过refresh token)。

```txt
1，客户端准备身份认证请求, 请求里包含所需的参数
2，客户端发送请求到授权服务器
3，授权服务器对最终用户进行身份认证
4，授权服务器获得最终用户的同意/授权
5，授权服务器把最终用户发送回客户端, 同时带着授权码
6，客户端使用授权码向Token端点请求一个响应
7，客户端接收到响应, 响应的body里面包含着ID_Token 和 Access_Token
8，客户端验证ID_Token, 并获得用户的一些身份信息.
```

### Implicit(简化)

不适合于长时间访问。

```txt
1，客户端准备身份认证请求, 请求里包含所需的参数
2，客户端发送请求到授权服务器
3，授权服务器对最终用户进行身份认证
4，授权服务器获得最终用户的同意/授权
5，授权服务器把最终用户发送回客户端, 同时带着ID_Token.
  如果也请求了Access_Token的话, 那么Access_Token也会一同返回.
6，客户端验证ID_Token, 并获得用户的一些身份信息.
```

### Hybrid Flow(混合)

适合于长时间的访问。

要求客户端应用可以安全的维护`secret`。

```txt
1，客户端准备身份认证请求, 请求里包含所需的参数
2，客户端发送请求到授权服务器
3，授权服务器对最终用户进行身份认证
4，授权服务器获得最终用户的同意/授权
5，授权服务器把最终用户发送回客户端, 同时带着授权码,
   根据响应类型的不同, 也可能还带着一个或者多个其它的参数.
6，客户端使用授权码向Token端点请求一个响应
7，客户端接收到响应, 响应的body里面包含着ID_Token 和 Access_Token
8，客户端验证ID_Token, 并获得用户的一些身份信息.
```

参考：

[OpenID Connect 协议入门指南](https://www.jianshu.com/p/be7cc032a4e9)

[Identity Server 4 预备知识 -- OpenID Connect 简介](https://www.cnblogs.com/cgzl/p/9231219.html)

[Identity Server 4 预备知识 -- OAuth 2.0 简介](https://www.cnblogs.com/cgzl/p/9221488.html)

[[认证授权] 4.OIDC（OpenId Connect）身份认证（核心部分）](https://www.cnblogs.com/linianhui/p/openid-connect-core.html)