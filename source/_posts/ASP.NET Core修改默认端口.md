---
title: ASP.NET Core修改默认端口
date: 2020-05-13 10:10:27
tags:
---

监听的 IP 可以选择本地回环地址，特定的 IP 以及任意 IP，分别是：

```
127.0.0.1 127.0.0.2 127.0.0.3…… 本地回环地址
192.168.120.102 特定的 IP
0.0.0.0 任意 IP
```

* 监听本地回环地址时，则访问仅限于本机应用程序，不需要管理员权限来添加防火墙配置。

* 本地计算机配置了反向代理服务器，推荐使用本地回环地址。

* 如果让服务对外公开提供，则需要设置为 `0.0.0.0` 任意 IP。

## UseUrls

```cs
public class Program
{
    public static void Main(string[] args)
    {
        CreateWebHostBuilder(args).Build().Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
               .UseUrls("http://*:8080")
            .UseStartup<Startup>();
}
```

## appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "urls": "http://*:5000",
  "AllowedHosts": "*"
}
```

参考：[https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/host/web-host?view=aspnetcore-3.1#override-configuration](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/host/web-host?view=aspnetcore-3.1#override-configuration)

## 命令行

使用命令行参数 `--urls` ：

http协议，监听IP 地址，监听端口

```cs
dotnet WebApplication1.dll --urls http://0.0.0.0:5000
```

## 设置环境变量

ASPNETCORE_URLS

```cs
environment=ASPNETCORE_URLS='http://0.0.0.0:5000'
```

参考：

[dotnet 命令](https://docs.microsoft.com/zh-cn/dotnet/core/tools/dotnet)

[ASP.NET Core Web 主机](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/host/web-host?view=aspnetcore-3.1)

[你需要知道的这几种 asp.net core 修改默认端口的方式](https://www.cnblogs.com/huangxincheng/p/9569133.html)

[如何设置 ASP.NET Core 程序监听的 IP 和端口](https://blog.walterlv.com/post/configure-urls-and-port-for-asp-dotnet.html)
