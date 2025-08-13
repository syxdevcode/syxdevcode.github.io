---
title: DotNetCore使用Docker部署
date: 2018-05-14 17:34:11
tags:
- Docker
- DotNetCore
- AWS
- Ubuntu
categories: 
- DotNetCore
---
# DotNetCore使用Docker部署

## docker安装（ubuntu16.04）

```docker
sudo apt-get update
sudo apt-get install docker-ce
```

## 拉取microsoft/dotnet镜像

```docker
ubuntu@ip-172-31-16-20:~$ sudo docker pull microsoft/dotnet
Using default tag: latest
latest: Pulling from microsoft/dotnet
cc1a78bfd46b: Pull complete
6861473222a6: Pull complete
7e0b9c3b5ae0: Pull complete
3ec98735f56f: Pull complete
8b3d41e9a6c9: Pull complete
92c2ea8451ce: Pull complete
e321ed356c13: Pull complete
Digest: sha256:ec26f9d34e3e8a85eb3f0d269170e5502bdee7d1ed46ddc1356d9ba6d245e6fa
Status: Downloaded newer image for microsoft/dotnet:latest
```

## 运行microsoft/dotnet镜像

使用`docker run <image>`可以启动镜像，通过指定参数`-it`以交互模式（进入容器内部）启动
`--rm` 退出时，删除容器。

```docker
//启动一个dotnet镜像
$ docker run --rm -it microsoft/dotnet

//创建项目名为docker.Web的.NET Core MVC项目
dotnet new mvc -n docker.web

//进入docker.web文件夹
cd docker.web

//启动.NET Core MVC项目
dotnet run
```

运行结果：

```docker
ubuntu@ip-172-31-16-20:~$ sudo docker run --rm -it microsoft/dotnet
root@b2ce07e9f5e5:/# dotnet new mvc -n docker.web
The template "ASP.NET Core Web App (Model-View-Controller)" was created successfully.
This template contains technologies from parties other than Microsoft, see https://aka.ms/template-3pn for details.

Processing post-creation actions...
Running 'dotnet restore' on docker.web/docker.web.csproj...
  Restoring packages for /docker.web/docker.web.csproj...
  Generating MSBuild file /docker.web/obj/docker.web.csproj.nuget.g.props.
  Generating MSBuild file /docker.web/obj/docker.web.csproj.nuget.g.targets.
  Restore completed in 1.69 sec for /docker.web/docker.web.csproj.
  Restoring packages for /docker.web/docker.web.csproj...
  Restore completed in 338.5 ms for /docker.web/docker.web.csproj.

Restore succeeded.

root@b2ce07e9f5e5:/# cd docker.web
root@b2ce07e9f5e5:/docker.web# dotnet run
warn: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[35]
      No XML encryptor configured. Key {392f49bb-6372-47a0-a34b-94b82a079812} may be persisted to storage in unencrypted form.
warn: Microsoft.AspNetCore.Server.Kestrel[0]
      Unable to bind to http://localhost:5000 on the IPv6 loopback interface: 'Error -99 EADDRNOTAVAIL address not available'.
Hosting environment: Production
Content root path: /docker.web
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

## 挂载源代码

### 宿主机安装.NET Core SDK

```linux
wget -q packages-microsoft-prod.deb https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get install apt-transport-https
sudo apt-get update
sudo apt-get install dotnet-sdk-2.1.200
```

安装完毕后，我们依次执行以下命令创建一个.NET Core MVC项目:

```linux
//回到根目录
$ cd $HOME

//创建demo文件夹
$ mkdir demo
$ cd demo

//创建项目名为docker.web的.NET Core MVC项目
sudo dotnet new mvc -n docker.web

//进入docker.web文件夹
cd docker.web

//启动.NET Core MVC项目
sudo dotnet run
```

`sudo lsof -i :5000`：查看5000端口信息

### 挂载宿主机项目到容器中

在启动Docker镜像时，Docker允许我们通过使用`-v`参数挂载宿主机的文件到容器的指定目录下。

```docker
// 命令中的`\`结合`Enter`键构成换行符，允许我们换行输入一个长命令。
$ sudo docker run --rm -it \
-v $HOME/dotnet/docker.web:/app \
microsoft/dotnet:latest
```

运行结果：

```linux
ubuntu@ip-172-31-16-20:~$ sudo docker run --rm -it \
> -v $HOME/dotnet/docker.web:/app \
> microsoft/dotnet:latest
root@4e0ae54991c4:/# ls
app  boot  etc   lib    media  opt   root  sbin  sys  usr
bin  dev   home  lib64  mnt    proc  run   srv   tmp  var
root@4e0ae54991c4:/# cd app
root@4e0ae54991c4:/app# ls
Controllers  Startup.cs                    appsettings.json   docker.web.csproj
Models       Views                         bin                obj
Program.cs   appsettings.Development.json  bundleconfig.json  wwwroot
root@4e0ae54991c4:/app# dotnet run
warn: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[35]
      No XML encryptor configured. Key {3600e0ea-f9cc-4c43-b75f-58665338c060} may be persisted to storage in unencrypted form.
warn: Microsoft.AspNetCore.Server.Kestrel[0]
      Unable to bind to http://localhost:5000 on the IPv6 loopback interface: 'Error -99 EADDRNOTAVAIL address not available'.
Hosting environment: Production
Content root path: /app
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

上面说到是以共享的形式，而不是容器拥有一份宿主机目录的拷贝，意味着，在宿主机上对目录的更改，会即时反应到容器中。但反过来，容器中对共享目录的更改，不会反应到宿主机上，不然就打破了容器具有的隔离特性。

### 使用Dockerfile部署

Dockerfile用来定义你将要在容器中执行的系列操作。我们来创建一个Dockerfile：

```linux
//确保进入我们创建的MVC项目目录中去
$ cd $HOME/demo/HelloDocker.Web

//使用touch命令创建Dockerfile
$ sudo touch Dockerfile

//使用vi命令编辑Dockerfile
sudo vim Dockerfile
```

填入以下内容：

```docker
FROM microsoft/dotnet:latest
WORKDIR /app
COPY . /app
RUN dotnet restore
EXPOSE 5000
ENV ASPNETCORE_URLS http://*:5000
ENTRYPOINT ["dotnet","run"]
```

`wq`:保存退出

``` docker
FROM：指定容器使用的镜像
WORKDIR：指定工作目录
COPY：复制当前目录（其中.即代表当前目录）到容器中的/app目录下
RUN：命令指定容器中执行的命令
EXPOSE：指定容器暴露的端口号
ENV：指定环境参数，上面用来告诉.NETCore项目在所有网络接口上监听5000端口
ENTRYPOINT：制定容器的入口点
```

使用`docker build -t <name> <path>`指令打包镜像:

```docker
docker build -t docker.web .
```

`.`：表示当前路径

运行结果：

```docker
ubuntu@ip-172-31-16-20:~/dotnet/docker.web$ sudo docker build -t docker.web .
Sending build context to Docker daemon  4.937MB
Step 1/7 : FROM microsoft/dotnet:latest
 ---> 2ac9a416f201
Step 2/7 : WORKDIR /app
Removing intermediate container 921079b99047
 ---> 9449738bc5a1
Step 3/7 : COPY . /app
 ---> b64089162379
Step 4/7 : RUN dotnet restore
 ---> Running in 0f13fc033144
  Restore completed in 76.09 ms for /app/docker.web.csproj.
  Restoring packages for /app/docker.web.csproj...
  Restore completed in 499.14 ms for /app/docker.web.csproj.
Removing intermediate container 0f13fc033144
 ---> 31a054861344
Step 5/7 : EXPOSE 5000
 ---> Running in 2a87ea69cb01
Removing intermediate container 2a87ea69cb01
 ---> 83e3c278ae44
Step 6/7 : ENV ASPNETCORE_URLS http://*:5000
 ---> Running in 535c7da1ced9
Removing intermediate container 535c7da1ced9
 ---> 4965bd215b81
Step 7/7 : ENTRYPOINT ["dotnet","run"]
 ---> Running in d023e6d8160b
Removing intermediate container d023e6d8160b
 ---> eb265d4f49cd
Successfully built eb265d4f49cd
Successfully tagged docker.web:latest
```

运行新打包的镜像，并通过`-p`参数映射容器的5000到宿主机的80端口，其中`-d`参数告诉docker以后台任务形式运行镜像,`--rm`退出时删除容器,`--name`指定容器名称。

```docker
sudo docker run --name dockerweb -d --rm -p 80:5000 docker.web
```

通过`curl -i http://localhost`验证。

```linux
ubuntu@ip-172-31-16-20:~$ curl -i http://localhost
HTTP/1.1 200 OK
Date: Tue, 15 May 2018 05:43:45 GMT
Content-Type: text/html; charset=utf-8
Server: Kestrel
Transfer-Encoding: chunked

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Home Page - docker.web</title>
```

至此，完成dotnetcore项目部署。

参考：

[http://www.cnblogs.com/sheng-jie/p/8107877.html#autoid-4-0-0](http://www.cnblogs.com/sheng-jie/p/8107877.html#autoid-4-0-0)

[http://www.cnblogs.com/savorboard/p/dotnetcore-docker.html](http://www.cnblogs.com/savorboard/p/dotnetcore-docker.html)