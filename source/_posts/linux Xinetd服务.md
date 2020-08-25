---
title: linux Xinetd服务
date: 2020-08-21 17:50:29
tags:
- Linux
- CentOS7
- Linux服务
categories:
- Linux服务
---

## Linux进程概念


## 简介

`xinetd` （ `extended internet daemon` ）是新一代的网络守护进程服务程序，又叫超级 `Internet` 服务器,常用来管理多种轻量级 `Internet` 服务。
xinetd提供类似于 `inetd+tcp_wrapper` 的功能，但是更加强大和安全。

基于 `xinetd` 的服务有启动管理和自启动管理之分，而且不管是启动管理还是自启动管理，都只有一种方法，相比独立的服务简单一些。

基于 `xinetd` 的服务没有自己独立的启动脚本程序，是需要依赖 `xinetd` 的启动脚本来启动的。`xinetd` 本身是独立的服务，所以 `xinetd` 服务自己的启动方法和独立服务的启动方法是一致的。







参考：

[什么是Xinetd](http://blog.chinaunix.net/uid-21411227-id-1826885.html)

[CentOS 7.3 Xinetd服务的安装与配置](https://blog.51cto.com/13525470/2060765)

[Linux之Xinetd服务介绍](https://blog.csdn.net/lzghxjt/article/details/83018710)

[Linux基于xinetd服务的管理方法详解](http://c.biancheng.net/view/1054.html)