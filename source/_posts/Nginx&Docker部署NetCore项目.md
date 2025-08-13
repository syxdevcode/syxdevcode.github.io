---
title: Nginx&Docker部署NetCore项目
date: 2018-05-15 13:48:24
tags:
- Docker
- DotNetCore
- Nginx
- AWS
- Ubuntu
categories: 
- DotNetCore
---
# Nginx&Docker部署NetCore项目

## Nginx运行在Docker

```docker
sudo docker container run \
  -d \
  -p 127.0.0.2:8080:80 \
  --rm \
  --name mynginx \
  nginx
```

上面命令下载并运行官方的 Nginx image，默认是最新版本（latest）。如果本机安装过以前的版本，请删掉重新安装，因为只有 1.13.9 才开始支持 server push。

上面命令的各个参数含义如下:

```docker
-d：在后台运行
-p ：容器的80端口映射到127.0.0.2:8080
--rm：容器停止运行后，自动删除容器文件
--name：容器的名字为mynginx
```

如果没有报错，就可以打开浏览器访问 127.0.0.2:8080 了。正常情况下，显示 Nginx 的欢迎页。

或者用curl显示：

```docker
ubuntu@ip-172-31-16-20:~/dotnet/conf$ curl http://127.0.0.1:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
ubuntu@ip-172-31-16-20:~/dotnet/conf$
```

然后，把这个容器终止，由于--rm参数的作用，容器文件会自动删除。

```docker
sudo docker container stop mynginx
```

## 拷贝配置

首先，把容器里面的 Nginx 配置文件拷贝到本地。

```docker
sudo docker container cp mynginx:/etc/nginx .
```

上面命令的含义是，把mynginx容器的/etc/nginx拷贝到当前目录。不要漏掉最后那个点。

执行完成后，当前目录应该多出一个nginx子目录。然后，把这个子目录改名为conf。

```docker
sudo mv nginx conf
```

容器终止:

```docker
sudo docker container stop mynginx
```

## 映射配置目录

```nginx
sudo docker container run \
  --name mynginx \
  --rm \
  --volume "$PWD/conf":/etc/nginx \
  -p 80:80 \
  -d \
  nginx
```

--volume "$PWD/conf":/etc/nginx表示把容器的配置目录/etc/nginx，映射到本地的conf子目录。

查看docker 容器日志：

```docker
sudo docker logs mynginx
```

## 映射证书

使用Let’s Encrypt生成的证书文件

```nginx
sudo docker container run \
  --name mynginx \
  --rm \
  --volume "$PWD/conf":/etc/nginx \
  -v /etc/letsencrypt:/etc/letsencrypt/ \
  -d \
  nginx
```

-v : 映射本地/etc/letsencrypt到容器/etc/letsencrypt目录。

## HTTPS 配置

配置`conf/conf.d/default.conf`文件，具体配置如下：

``` nginx
server {
        listen 80;
        server_name www.shiyx.top;
        return 301 https://www.shiyx.top$request_uri;
        server_tokens off;
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
            proxy_pass http://172.17.0.3:5000;
            proxy_http_version 1.1;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection keep-alive;
            proxy_set_header   Host $http_host;
            proxy_cache_bypass $http_upgrade;
        }
        location ~ /.well-known{
                allow all;
        }
}
```

查看docker ip地址

``` docker
sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-ID>
```

最后，启动一个新的Nginx容器。

** 注意运行目录**

```nginx
sudo docker container run \
  --name mynginx \
  --rm \
  --volume "$PWD/conf":/etc/nginx \
  -v /etc/letsencrypt:/etc/letsencrypt/ \
  -p 80:80 \
  -p 443:443 \
  -d \
  nginx
```

参考：
[http://www.ruanyifeng.com/blog/2018/02/nginx-docker.html](http://www.ruanyifeng.com/blog/2018/02/nginx-docker.html)

[https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-2.0&tabs=aspnetcore2x](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/linux-nginx?view=aspnetcore-2.0&tabs=aspnetcore2x)

[http://www.cnblogs.com/savorboard/p/dotnet-core-publish-nginx.html](http://www.cnblogs.com/savorboard/p/dotnet-core-publish-nginx.html)