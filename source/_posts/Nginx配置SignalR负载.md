---
title: Nginx配置SignalR负载
date: 2023-05-30 11:44:19
tags:
  - Docker Compose
  - Docker
  - Nginx
  - Ubuntu
  - Linux
  - SignalR
categories:
  - SignalR
---

```conf
upstream apiclusterhealthy{
    server 10.10.0.17:30000;
    server 10.10.0.19:30000;
    server 10.10.0.39:30000;

    ip_hash;
}

map $http_connection $connection_upgrade {
    "~*Upgrade" $http_connection;
    default keep-alive;
}

## server 内配置
    #
    location /cloud/hubs/ {
        proxy_pass   http://apiclusterhealthy;   #转发请求的地址
        rewrite      ^/cloud/(.*)$ /$1 break;

        # include      /etc/nginx/nginxconfig.io/proxy.conf;
        # Configuration for WebSockets
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_cache off;
        # WebSockets were implemented after http/1.0
        proxy_http_version 1.1;

        # Configuration for ServerSentEvents
        proxy_buffering off;

        # Configuration for LongPolling or if your KeepAliveInterval is longer than 60 seconds
        proxy_read_timeout 100s;

        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
```
<!--more-->

```ts
import * as SignalR from '@microsoft/signalr';
import { ElNotification } from 'element-plus';
import { getToken } from '/@/utils/axios-utils';

// 初始化SignalR对象
// WebSockets配置
const connection = new SignalR.HubConnectionBuilder()
    .configureLogging(SignalR.LogLevel.Information)
    .withUrl(`${import.meta.env.VITE_API_URL}/hubs/onlineUser?access_token=${getToken()}`, { transport: SignalR.HttpTransportType.WebSockets, skipNegotiation: true })
    .withAutomaticReconnect({
        nextRetryDelayInMilliseconds: () => {
            return 5000; // 每5秒重连一次
        },
    })
    .build();
connection.keepAliveIntervalInMilliseconds = 15 * 1000; // 心跳检测15s
connection.serverTimeoutInMilliseconds = 2 * 60 * 1000; // 超时时间2m
```

```cs
services.AddSignalR(hubOptions =>
    {
        hubOptions.EnableDetailedErrors = true;
        hubOptions.ClientTimeoutInterval= TimeSpan.FromMinutes(2);
        hubOptions.KeepAliveInterval = TimeSpan.FromMinutes(1);
    })
    .AddMessagePackProtocol()
    .AddStackExchangeRedis(options =>
    {
        options.Configuration.ChannelPrefix = signalRRedisOpt.Prefix;
        options.ConnectionFactory = async writer =>
        {
            var config = new ConfigurationOptions
            {
                AbortOnConnectFail = false,
                ServiceName = signalRRedisOpt.ServiceName,
                AllowAdmin = true,
                DefaultDatabase = signalRRedisOpt.DefaultDb,
                Password = signalRRedisOpt.Password
            };
            signalRRedisOpt.EndPoints.ForEach(o => config.EndPoints.Add(o));
            //config.SetDefaultPorts();
            var connection = await ConnectionMultiplexer.ConnectAsync(config, writer);
            connection.ConnectionFailed += (_, e) =>
            {
                "Connection to Redis failed.".LogError();
            };

            if (!connection.IsConnected)
            {
                "Did not connect to Redis.".LogError();
            }
            return connection;
        };
    });
```
