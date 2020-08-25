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

### 守护进程

Linux系统上提供服务的程序是由运行在后台的守护进程（Daemon）来执行。一个实际运行中的Linux系统一般会有多个这样的程序在运行。这些后台守护进程在系统开机后就运行了，并且在时刻地监听前台客户地服务请求，一旦客户发出了服务请求，守护进程便为它们提供服务。Windows系统中的守护进程被称为 `服务`。

按照服务类型，守护进程可以分为如下两类：

* 系统守护进程：如 `crond(周期任务)`、`rsyslogd(日志服务)`、`cpus`等；
* 网络守护进程：如 `sshd`、`httpd`、`xinetd（托管）`等。

**系统守护进程**

系统初始化进程是一个特殊的的守护进程，其PID为1，它是所有其他守护进程的父进程或者祖先进程。也就是说，系统上所有的守护进程都是由系统初始化进程进行管理的（如启动、停止等）。

在Linux的发展历史过程中，使用过3种Linux初始化系统。

SysVinit

为 UNIX System V 系统创建的；
RHEL/CentOS 5及之前的版本一直使用。

Upstart

由Ubuntu创建的；
RHEL/CentOS 6 使用Upstart。

Systemd

先进的初始化系统；
RHEL/CentOS 7使用Systemd。



独立启动的守护进程：stand-alone，每个特定服务都有单独的守护进程，这个处理单一服务的始终存在的进程就是独立启动的守护进程。
超级守护进程：多个服务统一由一个进程管理，该进程可以管理多个服务。
Xinetd：即extended internet daemon，是新一代的网络守护进程服务程序，又叫超级Internet服务器，常用来管理多种轻量级Internet服务。Xinetd提供类似于inetd+tcp_wrapper的功能，但是更加强大和安全。

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