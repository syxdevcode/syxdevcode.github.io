---
title: Nginx配置文件
date: 2021-09-08 14:19:23
tags:
- Nginx
- 安装部署
categories:
- Nginx
---

```conf
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    server {
        listen 80;
        server_name www.shiyx.top;
        return 301 https://www.shiyx.top$request_uri;
        server_tokens off;
    }
    upstream app_servers {
        server dockerwebapp:5000;
    }
    server {
        listen 443 ssl http2;
        ssl on;
        server_name www.shiyx.top;
        #$host该变量的值等于请求头中Host的值。如果Host无效时，那么就是处理该请求的server的名称。
        #permanent: 永久性重定向。请求日志中的状态码为301
        #nginx 对文档检测比较严格，所以if ( $host != 'www.csdn.com' ) 这些代码之间需要有空格隔开，不然会
        #报错：unknown directive “if($host!=”
        if ($host != 'www.shiyx.top' ){
                rewrite ^/(.*)$ https://www.shiyx.top/$1 permanent;
        }
        ssl_certificate /etc/letsencrypt/live/shiyx.top/fullchain.pem;

        ssl_certificate_key /etc/letsencrypt/live/shiyx.top/privkey.pem;

        #禁止在header中出现服务器版本，防止黑客利用版本漏洞攻击
        server_tokens off;
        # 设置ssl/tls会话缓存的类型和大小。如果设置了这个参数一般是shared，buildin可能会参数内存碎片，默认是none，
        #和off差不多，停用缓存。如shared:SSL:10m表示我>所有的nginx工作进程共享ssl会话缓存，
        #网介绍说1M可以存放约4000个sessions。
        ssl_session_cache    shared:SSL:1m;

        # 客户端可以重用会话缓存中ssl参数的过期时间，内网系统默认5分钟太短了，可以设成30m即30分钟甚至4h。
        ssl_session_timeout  5m;

        ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;

        # 选择加密套件，不同的浏览器所支持的套件（和顺序）可能会不同。
        # 这里指定的是OpenSSL库能够识别的写法，你可以通过 openssl -v cipher 'RC4:HIGH:!aNULL:!MD5'
        #（后面是你所指定的套件加密算法） 来看所支持算法。
        ssl_ciphers  HIGH:!aNULL:!MD5;

        # 设置协商加密算法时，优先使用我们服务端的加密套件，而不是客户端浏览器的加密套件。
        ssl_prefer_server_ciphers  on;

        # 查看代理程序的ip
        # sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-ID>
        location / {
            proxy_pass http://app_servers;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection keep-alive;
            proxy_set_header   Host $http_host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Host $server_name;
            proxy_cache_bypass $http_upgrade;
        }
        location ~ /.well-known{
                allow all;
        }
    }
}
```
