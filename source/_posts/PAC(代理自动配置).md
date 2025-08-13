---
title: PAC（代理自动配置）
date: 2022-05-10 11:25:53
tags:
- PAC
categories:
- 计算机基础
---

## 代理自动配置(Proxy Auto Config)

PAC（`Proxy Auto-Config`） 配置文件使用 Javascript 进行 URL 和代理服务器的描述。通常使用 `proxy.pac` 作为文件名， WPAD 的规范则使用 `wpad.dat` 作为 PAC 文件的文件名。

一个PAC文件包含一个JavaScript形式的函数 `FindProxyForURL(url, host)`。这个函数返回一个包含一个或多个访问规则的字符串。用户代理根据这些规则适用一个特定的代理器或者直接访问。当一个代理服务器无法响应的时候，多个访问规则提供了其他的后备访问方法。 浏览器在访问其他页面以前，首先访问这个PAC文件。PAC文件中的URL可能是手工配置的，也可能是通过网页的网络代理自发现协议（`Web Proxy Autodiscovery Protocol`）自动配置的。

## 自动化技术

现代的浏览器实现了几个级别的自动化；用户可以选择最适合他们需要的级别。下面的这些方法被普遍的实现：

* **手动代理配置** ：为所有的URLs规定一个主机名和端口作为代理。大多数浏览器允许用户规定一个域名的列表(例如 localhost)，访问这个列表里面的域名的时候不通过代理服务器。
* **代理自动配置(PAC)** ：规定一个指向PAC文件的URL，这个文件中包括一个JavaScript函数来确定访问每个URL时所选用的合适代理。这个方法更加适合需要几个不同代理配置的笔记本用户，或者有很多不同代理服务器的复杂的企业级设置。
* **网络代理自发现协议(WPAD)** ： 浏览器通过DHCP和DNS的查询来搜索PAC文件的位置。

<!--more-->
## PAC文件

要使用PAC，我们应当在一个网页服务器上发布一个PAC文件，并且通过在浏览器的代理链接设置页面输入这个PAC文件的URL或者通过使用WPAD协议告知用户代理去使用这个文件。

一个PAC文件是一个至少定义了一个JavaScript函数的文本文件。这个函数 `FindProxyForURL(url, host)` 有2个参数：url是一个对象的URL，host是一个由这个URL所派生的主机名。按照惯例，这个文件名字一般是 `proxy.pac.WPAD` 标准使用 `wpad.dat`。

虽然大多数客户端无论从HTTP请求返回的MIME类型是什么都能正确处理，但为了完整性和最佳的兼容性，我们应该设置网页服务器将这个文件的MIME类型声明为 `application/x-ns-proxy-autoconfig` 或者 `application/x-javascript-config` 。

PAC文件内容示例如下：

```js
function FindProxyForURL(url, host) {
   if (url== 'http://Her0in.org/') return 'DIRECT';
   if (shExpMatch(host, "*.wooyun.org")) return "DIRECT";
   if (host== 'wooyun.com') return 'SOCKS 127.1.1.1:8080';
   if (dnsResolve(host) == '10.0.0.100') return 'PROXY 127.2.2.2:8080;DIRECT';
   return 'DIRECT';
}
```

该文件定义了当用户访问 `http://Her0in.org/` 时，将不使用任何代理服务器直接（DIRECT）访问 URL。
也可以使用 `shExpMatch` 函数对 host 或者 url 进行匹配设置，`SOCKS 127.1.1.1:8080` 指定了使用 `127.1.1.1:8080` 的 SOCKS 代理进行 URL 的访问
`PROXY 127.2.2.2:8080;DIRECT` 指定了使用 `127.2.2.2:8080` 的 HTTP 代理进行 URL 的访问，如果连接 `127.2.2.2:8080` 的 HTTP 代理服务器失败，则直接（DIRECT）访问 URL。

本地搭建提供 WPAD 使用的 HTTP 代理服务器时，需要监听 80 端口，因为客户端浏览器默认会从 80 端口下载 PAC 文件，同时要将 PAC 文件的 MIME 类型设置为 `application/x-ns-proxy-autoconfig` 或 `application/x-javascript-config`。

## PAC文件的编码问题

有些浏览器，例如 `Firefox` 和 `Internet Explorer` 只支持系统默认的编码类型 的 PAC 文件，不支持Unicode编码的PAC文件，例如`UTF-8`编码的PAC文件。

参考：

[PAC（代理自动配置）](https://baike.baidu.com/item/PAC/16292100)