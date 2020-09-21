---
title: ASP.Net Core 3.1记录请求,响应
date: 2020-09-04 17:18:32
tags:
- CSharp
- DotNetCore
categories:
- DotNetCore
---

## 中间件

```cs
using Model.Base;
using Service.MonitorLog;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using System;
using System.Diagnostics;
using System.IO;

namespace Test
{
    public static class CustomLoggingMiddleware
    {
        public static void UseCustomLogging(this IApplicationBuilder app)
        {
            app.Use(async (context, next) =>
            {
                var requestContent = "";
                var contentType = context.Request.ContentType;
                context.Request.EnableBuffering();
                // 只接受json格式参数，防止文件流等参数超长
                if (!string.IsNullOrWhiteSpace(contentType) && contentType.Contains("json", StringComparison.OrdinalIgnoreCase))
                {
                    var requestReader = new StreamReader(context.Request.Body);
                    requestContent = await requestReader.ReadToEndAsync();
                }
                context.Request.Body.Position = 0;

                monitorlog ml = new monitorlog();

                var dt = DateTime.Now;

                ml.id = 0;
                ml.jkmc = context.Request.Path;
                ml.qqfs = context.Request.Method;
                ml.qqcs_txt = requestContent;

                Stream originalBody = context.Response.Body;
                try
                {
                    await using var memStream = new MemoryStream();
                    context.Response.Body = memStream;
                    await next();
                    memStream.Position = 0;
                    string responseBody = await (new StreamReader(memStream)).ReadToEndAsync();
                    ml.xyjg_txt = responseBody;

                    memStream.Position = 0;
                    await memStream.CopyToAsync(originalBody);
                }
                finally
                {
                    context.Response.Body = originalBody;
                }

                ml.statuscode = context.Response.StatusCode;
                if (ml.statuscode == 200)
                    await MonitorLog.SaveMonitorLog(ml);
            });
        }
    }
}
```

## 使用

在 `Startup.cs` 的 `Configure` 方法添加如下代码：

```cs
app.UseCustomLogging();
```