---
title: Docker & Docker Compose & Nginx 部署DotNetCore项目
date: 2018-05-17 08:51:19
tags:
- Docker Compose
- Nginx
- AWS
- Ubuntu
- Docker
- DotNetCore
categories: 
- Docker Compose
---
# Docker & Docker Compose & Nginx 部署DotNetCore项目

## Docker Compose简介

文档：[https://docs.docker.com/compose/](https://docs.docker.com/compose/)

`Compose` 项目是 Docker 官方的开源项目，负责实现对 `Docker` 容器集群的快速编排。从功能上看，跟 `OpenStack` 中的 `Heat` 十分类似。

其代码目前在 [https://github.com/docker/compose](https://github.com/docker/compose) 上开源。

`Compose` 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig。

通过第一部分中的介绍，我们知道使用一个 `Dockerfile` 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

`Compose` 恰好满足了这样的需求。它允许用户通过一个单独的 `docker-compose.yml` 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。

`Compose` 中有两个重要的概念：

`服务 (service)`：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。

`项目 (project)`：由一组关联的应用容器组成的一个完整业务单元，在 `docker-compose.yml` 文件中定义。

`Compose` 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。

`Compose` 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。

## Docker Compose安装

[https://docs.docker.com/compose/install/#install-compose](https://docs.docker.com/compose/install/#install-compose)

```docker
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
docker-compose version 1.21.2, build a133471
```

## 常用命令

### YAML配置命令

配置     |   说明
---      |  ---
build    | 指定 Dockerfile 所在的目录地址，用于构建镜像，并使用此镜像创建容器，比如上面配置的 `build: .`
command  | 容器的执行命令
dns      | 自定义 dns 服务器
expose   | 暴露端口配置，但不映射到宿主机，只被连接的服务访问
extends  | 对`docker-compose.yml`的扩展，配置在服务中
image    | 使用的镜像名称或镜像 ID
links    | 链接到其它服务中的容器（一般桥接网络模式使用）
net      | 设置容器的网络模式（四种：`bridge`, `none`, `container:[name or id]`和`host`）
ports    | 暴露端口信息，主机和容器的端口映射
volumes  | 数据卷所挂载路径设置

### Docker Compose 常用命令

命令 | 说明
---  | ---
docker-compose build  | 构建项目中的镜像，--force-rm：删除构建过程中的临时容器；--no-cache：不使用缓存构建；--pull：获取最新版本的镜像
docker-compose up -d  |  构建镜像、创建服务和启动项目，-d表示后台运行
docker-compose run ubuntu ls -d  | 指定服务上运行一个命令，-d表示后台运行
docker-compose logs  | 查看服务容器输出日志
docker-compose ps | 列出项目中所有的容器
docker-compose pause [service_name]  | 暂停一个服务容器
docker-compose unpause [service_name]  | 恢复已暂停的一个服务容器
docker-compose restart  | 重启项目中的所有服务容器（也可以指定具体的服务）
docker-compose stop | 停止运行项目中的所有服务容器（也可以指定具体的服务）
docker-compose start | 启动已经停止项目中的所有服务容器（也可以指定具体的服务）
docker-compose rm | 删除项目中的所有服务容器（也可以指定具体的服务），-f：强制删除（包含运行的）
docker-compose kill | 强制停止项目中的所有服务容器（也可以指定具体的服务）

## 编写一个docker-compose.yml

`dockers-compose.yml`文件要定义在我们项目的文件夹下。

整体目录结构：

```bash
|-- docker.web
  |--Dockerfile
|-- nginx
  |--nginx.conf
  |--Dockerfile
|--docker-compose.yml
```

```docker
version: '3'

services:
  dockerwebapp:
    container_name:dockerwebapp
    build:
      context: ./docker.web
      dockerfile: Dockerfile
    expose:
      - "5000"
    ports:
     - "5000:5000"
  reverse-proxy:
    container_name: reverse-proxy-nginx
    build:
      context: ./nginx
    volumes:
        - /etc/letsencrypt:/etc/letsencrypt/
    ports:
     - "80:80"
     - "443:443"
```

**docker.web下Dockerfile:**

```docker
FROM microsoft/dotnet:latest
WORKDIR /app
COPY . /app
RUN dotnet restore
EXPOSE 5000
ENV ASPNETCORE_URLS http://*:5000
ENTRYPOINT ["dotnet","run"]
```

**nginx下Dockerfile**

```sh
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
```

**nginx下nginx.conf配置**

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

其中要注意反向代理的配置：`server dockerwebapp:5000;`，其中ip部分直接指定的是docker-compose.yml中定义的第一个服务的名称dockerwebapp

## 启动compose

```sh
docker-compose up -d --build
```

运行结果：

```sh
ubuntu@ip-172-31-16-20:~/dotnet$ docker-compose up -d --build
Creating network "dotnet_default" with the default driver
Building dockerwebapp
Step 1/7 : FROM microsoft/dotnet:latest
 ---> 2ac9a416f201
Step 2/7 : WORKDIR /app
 ---> Using cache
 ---> 9449738bc5a1
Step 3/7 : COPY . /app
 ---> 63468e7420e1
Step 4/7 : RUN dotnet restore
 ---> Running in 24ecd554e076
  Restore completed in 75.73 ms for /app/docker.web.csproj.
  Restoring packages for /app/docker.web.csproj...
  Restore completed in 464.93 ms for /app/docker.web.csproj.
Removing intermediate container 24ecd554e076
 ---> cdc5f195aa36
Step 5/7 : EXPOSE 5000
 ---> Running in af5717cc39b8
Removing intermediate container af5717cc39b8
 ---> afe466d24f6d
Step 6/7 : ENV ASPNETCORE_URLS http://*:5000
 ---> Running in d151cad8058d
Removing intermediate container d151cad8058d
 ---> d4e8dadfe89c
Step 7/7 : ENTRYPOINT ["dotnet","run"]
 ---> Running in de41a97d97a8
Removing intermediate container de41a97d97a8
 ---> 05c1f1c48653
Successfully built 05c1f1c48653
Successfully tagged dotnet_dockerwebapp:latest
Building reverse-proxy
Step 1/2 : FROM nginx
 ---> ae513a47849c
Step 2/2 : COPY nginx.conf /etc/nginx/nginx.conf
 ---> 458d9128600f
Successfully built 458d9128600f
Successfully tagged dotnet_reverse-proxy:latest
Creating reverse-proxy-nginx ... done
Creating dockerwebapp        ... done
```

验证启动的服务：

```sh
sudo docker-compose ps
```

显示结果：

```sh
ubuntu@ip-172-31-16-20:~/dotnet$ docker-compose ps
       Name                 Command          State              Ports
--------------------------------------------------------------------------------
dockerwebapp          dotnet run             Up      5000/tcp
reverse-proxy-nginx   nginx -g daemon off;   Up      0.0.0.0:443->443/tcp,
                                                     0.0.0.0:80->80/tcp
```

**remove-orphans**

```sh
docker-compose down --rmi local --remove-orphans
```

## 删除tag为none镜像

```sh
docker images -a|grep none|awk '{print $3}'|xargs docker rmi
## 显示依赖关系
docker image inspect --format='{{.RepoTags}} {{.Id}} {{.Parent}}' $(docker image ls -q --filter since=b64089162379)
```

其中b64089162379为镜像ID。

参考：

[https://yeasy.gitbooks.io/docker_practice/content/compose/introduction.html](https://yeasy.gitbooks.io/docker_practice/content/compose/introduction.html)

[https://www.sep.com/sep-blog/2017/02/27/nginx-reverse-proxy-to-asp-net-core-separate-docker-containers/](https://www.sep.com/sep-blog/2017/02/27/nginx-reverse-proxy-to-asp-net-core-separate-docker-containers/)

[https://www.cnblogs.com/sheng-jie/p/8116276.html](https://www.cnblogs.com/sheng-jie/p/8116276.html)

[http://www.cnblogs.com/xishuai/p/docker-compose.html](http://www.cnblogs.com/xishuai/p/docker-compose.html)

[http://www.cnblogs.com/lsxqw2004/p/6709766.html](http://www.cnblogs.com/lsxqw2004/p/6709766.html)