---
title: Nginx配置负载
date: 2022-09-20 14:03:32
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

## 参数简介

```conf
# max_conns 最大连接数
upstream tomcats {
    server 10.10.0.103:8080 max_conns=20000;
    server 10.10.0.104:8080 max_conns=20000;
    server 10.10.0.105:8080 max_conns=20000;
}

# slow_start=60s  60s 之后该主机开始提供服务
# slow_start参数不能使用在hash radom load balaning中
# 如果在upstram中之有一台server则该参数无效
upstream tomcats {
    server 10.10.0.103:8080 weight=6 slow_start=60s;
    server 10.10.0.104:8080 weight=2;
    server 10.10.0.105:8080 weight=2;
}

# down 表示该主机不可用
upstream tomcats {
    server 10.10.0.103:8080 weight=6;
    server 10.10.0.104:8080 down;
}

# backup 表示当前服务器节点是备用机，之有在其他的所有服务都宕机以后，自己才会加入到集群当中
# backup参数不能使用在hash radom load balaning中
upstream tomcats {
    server 10.10.0.103:8080 backup;
    server 10.10.0.104:8080 weight=1;
    server 10.10.0.105:8080 weight=1;
}
```

<!--more-->
## 负载配置

```conf
# 1，默认采用轮询
upstream tomcats {
    server 10.10.0.103:8080;
    server 10.10.0.104:8080;
    server 10.10.0.105:8080;
}

# 2，weight 根据权重来负载
upstream tomcats {
    server 10.10.0.103:8080 weight=6;
    server 10.10.0.104:8080 weight=2;
    server 10.10.0.105:8080 weight=2;
}

# 3，ip_hash
# ip_hash可以保证用户访问可以请求到上游的服务器中固定的服务器，前提是用户ip没有发生更改
# 使用ip_hash 注意，不能把后台服务器直接移除，只能标记为down
upstream tomcats {
    ip_hash;

    server 10.10.0.103:8080;
    server 10.10.0.104:8080 down;
    server 10.10.0.105:8080;
}

# 4，url_hash
# url_hash 每次根据请求的url地址hash后访问到固定的服务器节点
upstream tomcats {
    # url hash
    hash $request_uri;
    server 10.10.0.103:8080;
    server 10.10.0.104:8080;
    server 10.10.0.105:8080;
}

# least_conn 最少连接数
# least_conn指令实际的含义就是，选取活跃连接数与权重weight的比值最小者为下一个处理请求的server。
# 当然，上一次已选的server和已达到最大连接数的server照例不在选择的范围。
upstream tomcats {
    least_conn 

    server 10.10.0.103:8080;
    server 10.10.0.104:8080;
    server 10.10.0.105:8080;
}
```

## 实例

```conf
upstream apicluster{
    server 10.10.0.110:30008;
    server 10.10.0.111:30008;
    server 10.10.0.185:30008;
    server 10.10.0.186:30008;
}

server {
    listen              443;
    # listen              [::]:443 http2;
    server_name         example.com;
    root                /var/www/example.com/public;

    # SSL
    # ssl_certificate     /etc/nginx/ssl/example.com.crt;
    # ssl_certificate_key /etc/nginx/ssl/example.com.key;

    # security
    include             /etc/nginx/nginxconfig.io/security.conf;

    # logging
    access_log          /var/log/nginx/example.com.access.log;
    error_log           /var/log/nginx/example.com.error.log warn;
  
    # index.html fallback
    location / {
    #   root    /var/www/example.com/public;
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
    # WebSocket
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
    server_name example.com;
    return      301 https://example.com$request_uri;
}
```
