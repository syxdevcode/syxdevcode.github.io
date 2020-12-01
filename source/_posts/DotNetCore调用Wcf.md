---
title: DotNetCore调用Wcf
date: 2020-12-01 11:31:57
tags:
- DotNetCore
- Net5
- Wcf
categories: 
- Wcf
---

## 安装dotnet-svcutil

```cs
dotnet tool install --global dotnet-svcutil
```

## 生成

```cs
dotnet-svcutil --outputFile Test1Information.cs -n "http://tempuri.org/,API.ServiceReference.Services1"  --outputDir Services1 http://172.2.22.12/FRXXServices/Services1.asmx
dotnet-svcutil --outputFile Test2Services.cs -n "http://server.test.com,API.ServiceReference.Services2" --outputDir Services2 http://172.2.22.12:8000/services/Services2?WSDL
```

参考：

[https://docs.microsoft.com/en-us/dotnet/core/additional-tools/dotnet-svcutil-guide?tabs=dotnetsvcutil2x](https://docs.microsoft.com/en-us/dotnet/core/additional-tools/dotnet-svcutil-guide?tabs=dotnetsvcutil2x)