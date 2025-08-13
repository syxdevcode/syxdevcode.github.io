---
title: Nginx代理Minio
date: 2022-11-10 11:42:22
tags:
  - Docker Compose
  - Docker
  - RockyLinux
  - MinIo
  - Linux
  - Nginx
categories:
  - MinIo
---

目录结构：

```sh
├── dhparam.pem
├── fastcgi_params
├── logs
│   ├── access.log
│   ├── error.log
│   ├── limsweb.access.log
│   ├── limsweb.error.log
├── mime.types
├── nginx.conf
├── nginxconfig.io
│   ├── general.conf
│   ├── proxy.conf
│   └── security.conf
├── scgi_params
├── sites-enabled
│   └── limsweb.conf
├── ssl
│   ├── ilims.mylims.com.key
│   ├── ilims.mylims.com_bundle.crt
│   └── ilims.mylims.com_bundle.pem
├── uwsgi_params
└── www
    └── limsweb
        └── public
```

注意 general.conf 中需要注释掉 拦截 assets, media的配置:

```conf
# location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
#    expires    7d;
#    access_log off;
# }
```

<!--more-->

完整配置：

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
        
        location ~* \.(?:css(\.map)?|js(\.map)?|jpe?g|png|gif|ico|cur|heic|webp|tiff?|mp3|m4a|aac|ogg|midi?|wav|mp4|mov|webm|mpe?g|avi|ogv|flv|wmv)$ {
            expires    7d;
            access_log off;
        }
    }

    #代理后端接口
    location /api/ {
        proxy_pass   http://apicluster;   #转发请求的地址
        rewrite      ^/api/(.*)$ /$1 break;
        include      /etc/nginx/nginxconfig.io/proxy.conf;
    }

    # 以下为 minio 配置
    # To allow special characters in headers
    ignore_invalid_headers off;
    # Allow any size file to be uploaded.
    # Set to a value such as 1000m; to restrict file size to a specific value
    client_max_body_size 0;
    # To disable buffering
    proxy_buffering off;

    # 代理minio
    location /s3/ {
       rewrite      ^/s3/(.*)$ /$1 break;
              
       # Proxy headers
       proxy_set_header Upgrade           $http_upgrade;
       proxy_set_header Connection        $connection_upgrade;
       # proxy_set_header Host              $http_host;
       proxy_set_header Host              $host;
       proxy_set_header X-Real-IP         $remote_addr;
       proxy_set_header Forwarded         $proxy_add_forwarded;
       proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Forwarded-Host  $host;
       
       proxy_hide_header X-Powered-By;
       
       # Proxy timeouts
       proxy_connect_timeout              60s;
       proxy_send_timeout                 60s;
       proxy_read_timeout                 60s;

       # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
       proxy_http_version    1.1;
       proxy_cache_bypass   $http_upgrade;
       chunked_transfer_encoding  off;
       
       proxy_pass http://10.10.0.103:9100; # 转发请求的地址
    }
    #  minio 配置结束

    location /limscap/ { 
        return 404; 
    }

    # websocket
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

# HTTP redirect
server {
    listen      80;
    listen      [::]:80;
    server_name localhost;
    return      301 https://example.com$request_uri;
}
```
