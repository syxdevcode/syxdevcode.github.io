---
title: 卸载DotNetCore
date: 2018-05-03 16:59:47
tags:
- Linux
- CentOS7
- DotNetCore
categories: 
- DotNetCore
---
# 卸载DotNetCore

查找dotnet命令：

``` linux
which dotnet
```

查找dotnet相关的rpm包

``` linux
rpm -qa|grep dotnet
```

移除相关包：

``` linux
rpm -e dotnet-runtime-deps-2.1.0-preview2-26406-04-2.1.0_preview2_26406_04-1.x86_64
```