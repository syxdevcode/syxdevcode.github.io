---
title: DotNet6-Docker链接Sql-Server问题
date: 2022-09-06 14:06:34
tags:
  - Sql Server
  - DotNetCore
  - Docker
  - 命名实例
categories:
  - DotNetCore
---

## Sql Server 命名实例设置

默认安装的命名实例，需要单独设置端口号。

![Snipaste_2022-09-06_14-21-41.png](/img1/Snipaste_2022-09-06_14-21-41.png)

设置成功后，需要重启服务。

<!--more-->

## DotNet 链接字符串设置

使用 `SqlSugarCore` 版本：5.1.2.6，连接测试：

```cs

//创建数据库对象
SqlSugarClient db = new SqlSugarClient(new ConnectionConfig()
{
    // 成功
    //ConnectionString = @"Encrypt=True;TrustServerCertificate=True;Persist Security Info=False; User ID=test; Password=1234567; Initial Catalog=ConfigCenter; Server=10.10.0.36;",

    // 成功
    //ConnectionString = @"Encrypt=True;TrustServerCertificate=True;Persist Security Info=False; User ID=sa; Password=1234567; Initial Catalog=ConfigCenter; Server=10.10.0.36;",

    // 成功
    // ConnectionString = @"Encrypt=True;TrustServerCertificate=True;Persist Security Info=False;Initial Catalog=LimsBase;User Id=sa;Password=1234567;Server=10.10.0.36;",

    // 成功
    //ConnectionString = @"Encrypt=True;TrustServerCertificate=True;Persist Security Info=False;Initial Catalog=LimsBase;User Id=sa;Password=123123@$321!;Data Source=10.10.0.36,1435;",

    // 成功
    ConnectionString = "Server=10.10.0.36,1435;Database=LimsBase;User=sa;Password=123123@$321!;MultipleActiveResultSets=True;",

    DbType = DbType.SqlServer,
    IsAutoCloseConnection = true
});

//调试SQL事件，可以删掉 (要放在执行方法之前)
db.Aop.OnLogExecuting = (sql, pars) =>
{
    Console.WriteLine(sql); //输出sql,查看执行sql 性能无影响
};
```

## Dockerfile 设置

SQL Server 2016、SQL Server 2017 和 SQL Server 2019 支持 TLS 1.2，无需设置。

低版本 SQL Server 默认不支持 TLS 1.2（可以安装补丁包，详情：[KB3135244 - TLS 1.2 对 Microsoft SQL Server](https://support.microsoft.com/zh-cn/topic/kb3135244-tls-1-2-%E5%AF%B9-microsoft-sql-server-e4472ef8-90a9-13c1-e4d8-44aad198cdbe#:~:text=SQL%20Server%202016%E3%80%81SQL%20Server%202017%20%E5%92%8C%20SQL%20Server,%E5%BB%BA%E8%AE%AE%E5%8D%87%E7%BA%A7%E5%88%B0%20TLS%201.2%20%E4%BB%A5%E5%AE%89%E5%85%A8%E9%80%9A%E4%BF%A1%E3%80%82%20%E9%87%8D%E8%A6%81%EF%BC%9A%E6%9C%AA%E9%92%88%E5%AF%B9%20Microsoft%20TDS%20%E5%AE%9E%E7%8E%B0%E6%8A%A5%E5%91%8A%E4%BB%BB%E4%BD%95%E5%B7%B2%E7%9F%A5%E6%BC%8F%E6%B4%9E%E3%80%82)），但是在 Docker 映像容器、Unix 客户端或 Windows 客户端等客户端环境中，其中 TLS 1.2 是支持的最低 TLS 协议，所以需要设置 TLS 版本。

```Dockerfile
#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/runtime:6.0 AS base

# 设置TLS版本
RUN sed -i 's/DEFAULT@SECLEVEL=2/DEFAULT@SECLEVEL=1/g' /etc/ssl/openssl.cnf
RUN sed -i 's/MinProtocol = TLSv1.2/MinProtocol = TLSv1/g' /etc/ssl/openssl.cnf
RUN sed -i 's/DEFAULT@SECLEVEL=2/DEFAULT@SECLEVEL=1/g' /usr/lib/ssl/openssl.cnf
RUN sed -i 's/MinProtocol = TLSv1.2/MinProtocol = TLSv1/g' /usr/lib/ssl/openssl.cnf
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["ConsoleApp1/ConsoleApp1.csproj", "ConsoleApp1/"]
RUN dotnet restore "ConsoleApp1/ConsoleApp1.csproj"
COPY . .
WORKDIR "/src/ConsoleApp1"
RUN dotnet build "ConsoleApp1.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "ConsoleApp1.csproj" -c Release -o /app/publish /p:UseAppHost=false

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "ConsoleApp1.dll"]
```

参考：

[Sql 客户端疑难解答指南](https://docs.microsoft.com/zh-cn/sql/connect/ado-net/sqlclient-troubleshooting-guide?view=sql-server-ver15#possible-reasons-and-solutions)

[A network-related or instance-specific error occurred while establishing a connection to SQL Server](https://docs.microsoft.com/en-us/troubleshoot/sql/connect/network-related-or-instance-specific-error-occurred-while-establishing-connection#default-instance-of-sql-server)

[在与 SQL Server 2019 建立连接时出现与网络相关的或特定于实例的错误。未找到或无法访问服务器](https://blog.csdn.net/qq_43724983/article/details/105777602)

[SqlClient driver support lifecycle](https://docs.microsoft.com/zh-cn/sql/connect/ado-net/sqlclient-driver-support-lifecycle?view=sql-server-ver15)

[KB3135244 - TLS 1.2 对 Microsoft SQL Server](https://support.microsoft.com/zh-cn/topic/kb3135244-tls-1-2-%E5%AF%B9-microsoft-sql-server-e4472ef8-90a9-13c1-e4d8-44aad198cdbe#:~:text=SQL%20Server%202016%E3%80%81SQL%20Server%202017%20%E5%92%8C%20SQL%20Server,%E5%BB%BA%E8%AE%AE%E5%8D%87%E7%BA%A7%E5%88%B0%20TLS%201.2%20%E4%BB%A5%E5%AE%89%E5%85%A8%E9%80%9A%E4%BF%A1%E3%80%82%20%E9%87%8D%E8%A6%81%EF%BC%9A%E6%9C%AA%E9%92%88%E5%AF%B9%20Microsoft%20TDS%20%E5%AE%9E%E7%8E%B0%E6%8A%A5%E5%91%8A%E4%BB%BB%E4%BD%95%E5%B7%B2%E7%9F%A5%E6%BC%8F%E6%B4%9E%E3%80%82)
