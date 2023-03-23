---
title: OpenResty实战之Docker-Compose部署
date: 2023-03-11 14:39:01
tags:
  - OpenResty
  - Docker Compose
  - Docker
  - Ubuntu
  - RockyLinux
  - Linux
  - Nginx
categories:
  - OpenResty
---

```yml
version: "3.1"

services:
  nginx:
    image: nginx:1.22.0
    container_name: nginx
    restart: unless-stopped
    networks:
      - net0
    volumes:
      - /opt/nginx/www:/var/www
      - /opt/nginx/logs:/var/log/nginx
      - /opt/nginx:/etc/nginx
    ports:
      - 31000:443
    environment:
      - NGINX_PORT=80
      - TZ=Asia/Shanghai
    privileged: true
networks:
  net0:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 10.10.12.0/24
        gateway: 10.10.12.1

networks:
  default:
    external:
      name: docker_compose_net
```

```yml
version: "3"

networks:
  default:
    external:
      name: docker_compose_net

services:
  minio:
    restart: unless-stopped
    image: minio/minio
    container_name: minio
    ports:
      - "9100:9000" # api 端口
      - "9190:9090" # 控制台端口
    networks:
      - net0
    volumes:
      - /opt/minio/data:/data
    privileged: true
    environment:      
      MINIO_ACCESS_KEY: admin    #管理后台用户名
      MINIO_SECRET_KEY: admin123 #管理后台密码，最小8个字符
    command: server --console-address ':9090' /data  #指定容器中的目录 /data

```


```conf
user                 nginx;
pid                  /var/run/nginx.pid;
worker_processes     auto;
worker_rlimit_nofile 65535;

# Enables the use of JIT for regular expressions to speed-up their processing.
pcre_jit on;

events {
    multi_accept       on;
    worker_connections 65535;
}

http {
    charset                utf-8;
    sendfile               on;
    tcp_nopush             on;
    tcp_nodelay            on;
    server_tokens          off;
    types_hash_max_size    2048;
    types_hash_bucket_size 64;
    client_max_body_size   16M;

    # MIME
    include                mime.types;
    default_type           application/octet-stream;

    # Enables or disables the use of underscores in client request header fields.
    # When the use of underscores is disabled, request header fields whose names contain underscores are marked as invalid and become subject to the ignore_invalid_headers directive.
    # underscores_in_headers off;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    map $time_iso8601 $logdate {
      '~^(?<ymd>\d{4}-\d{2}-\d{2})' $ymd;
      default                       'date-not-found';
    }

    access_log logs/access-$logdate.log main;
    open_log_file_cache max=10;

    #access_log  logs/access.log  main;

        # Log in JSON Format
        # log_format nginxlog_json escape=json '{ "timestamp": "$time_iso8601", '
        # '"remote_addr": "$remote_addr", '
        #  '"body_bytes_sent": $body_bytes_sent, '
        #  '"request_time": $request_time, '
        #  '"response_status": $status, '
        #  '"request": "$request", '
        #  '"request_method": "$request_method", '
        #  '"host": "$host",'
        #  '"upstream_addr": "$upstream_addr",'
        #  '"http_x_forwarded_for": "$http_x_forwarded_for",'
        #  '"http_referrer": "$http_referer", '
        #  '"http_user_agent": "$http_user_agent", '
        #  '"http_version": "$server_protocol", '
        #  '"nginx_access": true }';
        # access_log /dev/stdout nginxlog_json;

    # See Move default writable paths to a dedicated directory (#119)
    # https://github.com/openresty/docker-openresty/issues/119
    client_body_temp_path /var/run/openresty/nginx-client-body;
    proxy_temp_path       /var/run/openresty/nginx-proxy;
    fastcgi_temp_path     /var/run/openresty/nginx-fastcgi;
    uwsgi_temp_path       /var/run/openresty/nginx-uwsgi;
    scgi_temp_path        /var/run/openresty/nginx-scgi;

    # Logging
    access_log             /var/log/nginx/access.log;
    error_log              /var/log/nginx/error.log warn;

    # SSL
    ssl_session_timeout    1d;
    ssl_session_cache      shared:SSL:10m;
    ssl_session_tickets    off;

    # Diffie-Hellman parameter for DHE ciphersuites
    ssl_dhparam            /etc/nginx/dhparam.pem;

    # Mozilla Intermediate configuration
    ssl_protocols          TLSv1.2 TLSv1.3;
    ssl_ciphers            ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;

    # OCSP Stapling
    # ssl_stapling           on;
    # ssl_stapling_verify    on;
    resolver               1.1.1.1 1.0.0.1 8.8.8.8 8.8.4.4 208.67.222.222 208.67.220.220 valid=60s;
    resolver_timeout       2s;

    # Connection header for WebSocket reverse proxy
    map $http_upgrade $connection_upgrade {
        default upgrade;
        ""      close;
    }

    map $remote_addr $proxy_forwarded_elem {

        # IPv4 addresses can be sent as-is
        ~^[0-9.]+$        "for=$remote_addr";

        # IPv6 addresses need to be bracketed and quoted
        ~^[0-9A-Fa-f:.]+$ "for=\"[$remote_addr]\"";

        # Unix domain socket names cannot be represented in RFC 7239 syntax
        default           "for=unknown";
    }

    map $http_forwarded $proxy_add_forwarded {

        # If the incoming Forwarded header is syntactically valid, append to it
        "~^(,[ \\t]*)*([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?(;([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?)*([ \\t]*,([ \\t]*([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?(;([!#$%&'*+.^_`|~0-9A-Za-z-]+=([!#$%&'*+.^_`|~0-9A-Za-z-]+|\"([\\t \\x21\\x23-\\x5B\\x5D-\\x7E\\x80-\\xFF]|\\\\[\\t \\x21-\\x7E\\x80-\\xFF])*\"))?)*)?)*$" "$http_forwarded, $proxy_forwarded_elem";

        # Otherwise, replace it
        default "$proxy_forwarded_elem";
    }
    
    # Load configs
    include                /etc/nginx/conf.d/*.conf;

    # Don't reveal OpenResty version to clients.
    server_tokens off;
}
```


