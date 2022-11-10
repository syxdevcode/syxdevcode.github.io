---
title: Nginx配置负载实例
date: 2022-09-20 19:18:43
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

例如：

配置 upstream 的 location 代码：注意添加了 `~*`

```conf
    #代理后端接口
    location ~* /api/ {
        proxy_pass   http://apicluster;   #转发请求的地址
        rewrite      ^/api/(.*)$ /$1 break;
        include      /etc/nginx/nginxconfig.io/proxy.conf;
    }
```

单节点 location 代码：

```conf
    #代理后端接口
    location /api/ {
        proxy_pass https://10.10.0.106:8082;   #转发请求的地址
        rewrite ^/api/(.*)$ /$1 break;
        include      /etc/nginx/nginxconfig.io/proxy.conf;
    }
```

<!--more-->

## 实例代码1

```conf
upstream apicluster{
    server 10.10.0.37:30008;
    server 10.10.0.39:30008;
    server 10.10.0.183:30008;
    server 10.10.0.184:30008;
}

server {
    listen              443;    
    # listen              443 ssl http2; # 注意：加上证书后需要这样配置
    # listen              [::]:4433 http2;
    server_name         10.10.0.106;

    # SSL
    # ssl_certificate     /etc/nginx/ssl/mysite.com.crt;
    # ssl_certificate_key /etc/nginx/ssl/mysite.com.key;

    # security
    include            /etc/nginx/nginxconfig.io/security.conf;

    # logging
    access_log          /var/log/nginx/mysite.access.log;
    error_log           /var/log/nginx/mysite.error.log warn;
   
    #代理后端接口
    location ~* /api/ {
        proxy_pass   http://apicluster;   #转发请求的地址
        include      /etc/nginx/nginxconfig.io/proxy.conf;
    }
    # WebSocket
    location ~* /hubs/ {
        proxy_pass http://apicluster;   #转发请求的地址
        include      /etc/nginx/nginxconfig.io/proxy.conf;
    }

    location ~* / {
        proxy_pass http://10.10.0.106:8080;
        include      /etc/nginx/nginxconfig.io/proxy.conf;
    }

    location @router {
        rewrite ^.*$ /index.html last;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

    # additional config
    include /etc/nginx/nginxconfig.io/general.conf;
}
```

## 实例代码2

```conf
upstream apicluster{
    server 10.10.0.117:30008;
    server 10.10.0.119:30008;
}

server {
    listen              443 ssl http2;
    listen              [::]:433 http2;
    server_name         ilims.mylims.com;
    root                /var/www/limsweb/public;

    charset utf-8;

    # SSL
    ssl_certificate     /etc/nginx/ssl/ilims.mylims.com_bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/ilims.mylims.com.key;

    # security
    include             /etc/nginx/nginxconfig.io/security.conf;
    
    # logging
    access_log          /var/log/nginx/limsweb.access.log;
    error_log           /var/log/nginx/limsweb.error.log warn;

    # index.html fallback
    location / {
            # root    /var/www/limsweb/public;
            try_files $uri $uri/ @router;
            index  index.html index.htm;
            error_page 405 =200 http://$host$request_uri;
    }
    #代理后端接口
    location /api/ {
        proxy_pass   http://apicluster;   #转发请求的地址
        rewrite      ^/api/(.*)$ /$1 break;
        include      /etc/nginx/nginxconfig.io/proxy.conf;
    }

    location /limscap/ {
        return 404;
    }

    #
    location /hubs/ {
        proxy_pass http://apicluster;   #转发请求的地址
        include      /etc/nginx/nginxconfig.io/proxy.conf;
    }
    location @router {
        rewrite ^.*$ /index.html last;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

    # additional config
    include /etc/nginx/nginxconfig.io/general.conf;
}
```
