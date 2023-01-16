---
title: 配置Nginx日志分割
date: 2023-01-16 15:59:22
tags:
  - Docker Compose
  - Docker
  - Ubuntu
  - RockyLinux
  - Linux
  - Nginx
categories:
  - Nginx
---

`nginx.conf` 配置：

```sh
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for"';

map $time_iso8601 $logdate {
  '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
  default                       'date-not-found';
}

access_log logs/access-$logdate.log main;
open_log_file_cache max=10;
```

参考：

[简单搞定Nginx日志分割](https://jingsam.github.io/2019/01/15/nginx-access-log.html)