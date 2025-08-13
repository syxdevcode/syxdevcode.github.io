---
title: win10安装OpenSSL
date: 2017-05-25 15:32:39
tags:
- Windows10
- OpenSSL
- 加密
- 安装部署
categories:
- OpenSSL
---

## 下载安装

openssl官网：[https://www.openssl.org/](https://www.openssl.org/ "openssl官网")

openssl下载地址：[http://slproweb.com/products/Win32OpenSSL.html](http://slproweb.com/products/Win32OpenSSL.html "openssl下载地址")

下载`Win64 OpenSSL v1.1.0e`(win10 64位)，点击依次点击下一步，即可完成安装；

## 设置环境变量

找到“系统->高级系统设置->环境变量”窗口里面的系统变量；

添加变量名为“OPENSSL_HOME”，变量值：“C:\OpenSSL-Win64\bin”（根据本机openssl安装目录配置），然后在变量名为“Path”中添加“%OPENSSL_HOME%”

如图：

![配置1](/img/openssl环境变量配置1.png)

![配置2](/img/openssl环境变量配置2.png)

注意：设置完环境变量后的cmd窗口必须重新打开，设置的环境变量才会起作用，否则依然提示命令不可用。