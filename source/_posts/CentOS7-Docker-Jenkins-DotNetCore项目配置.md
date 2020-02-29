---
title: CentOS7 & Docker & Jenkins & DotNetCore项目配置
date: 2018-05-08 13:54:33
tags: 
- Linux
- Jenkins
- CentOS7
- Docker
categories: 
- Jenkins
---
# CentOS7 & Docker & Jenkins & DotNetCore项目配置

## GitHub配置

### SercretText

注：此处需要一个对项目有写权限的账户

进入SGithub --> Setting --> Developer settings --> Personal access tokens

![1](/img/QQ截图20180508141541.png)

![2](/img/QQ截图20180508141948.png)

![3](/img/QQ截图20180508142138.png)

自己先保存此token，如果丢失，之后再也无法找到这个token。

## 配置GitHub Plugin

系统管理 --> 系统设置 --> GitHub --> GitHub Plugin

配置github用户名，邮箱

![4](/img/QQ截图20180508144549.png)

## 配置Jenkins

### 新建任务：

![5](/img/QQ截图20180508145651.png)

### General(通常)

![6](/img/QQ截图20180508145841.png)

### 源码管理

![7](/img/QQ截图20180508145941.png)

点击Add：

![8](/img/QQ截图20180508150102.png)

### 构建触发器：

`H/2 * * * *`

具体参考：

[https://blog.csdn.net/brave_insist/article/details/78434877](https://blog.csdn.net/brave_insist/article/details/78434877)

``` scm
每15分钟构建一次：H/15 * * * *   或*/5 * * * *

每天8点构建一次：0 8 * * *

每天8点~17点，两小时构建一次：0 8-17/2 * * *

周一到周五，8点~17点，两小时构建一次：0 8-17/2 * * 1-5

每月1号、15号各构建一次，除12月：H H 1,15 1-11 *

*/5 * * * * （每5分钟检查一次源码变化）

0 2 * * * （每天2:00 必须build一次源码）


每个部分代表的含义以及取值范围

分钟，取值范围（0~59）：若其他值不做设定，则表示每个设定的分钟都会构建
如：5 * * * * ，表示每个小时的第5分钟都会构建一次

小时，取值范围（0~23）：若其他值不做设定，则表示每个设定小时的每分钟都会构建
如：* 5 * * * ，表示在每天5点的时候，一小时内每一分钟都会构建一次

日期，取值范围（1~31）：若其他值不做设定，则表示每个月的那一天每分钟都会构建一次
如：* * 5 * *，表示在每个月5号的时候，0点开始每分钟构建一次

月份，取值范围（1~12）：若其他值不做设定，则表示每年的那个月每分钟都会构建一次
如：* * * 5 *，表示在每年的5月份，1号0点开始每分钟构建一次

星期，取值范围（0 ~ 7）：若其他值不做设定，则表示每周的那一天几每分钟都会构建一次
如：* * * * 5，表示每周五0点开始每分钟构建一次
```

![9](/img/QQ截图20180508150645.png)

### 构建环境

![10](/img/QQ截图20180508151257.png)

点击Add：

![10](/img/QQ截图20180508152137.png)

![11](/img/QQ截图20180508152518.png)

### 构建

![12](/img/QQ截图20180508152609.png)

``` bash
#!/bin/bash
# 获取短版本号

GITHASH=`git rev-parse --short HEAD`
echo ---------------Remove-Orphans------------------
docker-compose -f ./docker-compose.yml -f ./docker-compose.override.yml  -p dotnetcoreapp1 down --rmi local --remove-orphans
echo ------------------Config-----------------------
docker-compose -f ./docker-compose.ci.build.yml -p dotnetcoreapp1 config
echo ------------------Build------------------------
docker-compose -f ./docker-compose.ci.build.yml -p dotnetcoreapp1 up --build
echo ---------------Publishing...------------------
docker-compose -f "./docker-compose.yml" -f "./docker-compose.override.yml"  -p dotnetcoreapp1 up -d --build
echo ---------------Clear-Images...------------------
docker rmi -f $(docker images -f "dangling=true" -q)
echo ---------------Clear-Containers...------------------
docker rm dotnetcoreapp1_ci-build_1
```

命令说明：

``` docker
-f, --file FILE 指定使用的 Compose 模板文件，默认为 docker-compose.yml，可以多次指定。

-p, --project-name NAME 指定项目名称，默认将使用所在目录名称作为项目名。
```

修改docker-compose.override.yml文件：

``` docker
version: '3'

services:
  dotnetcoreapp:
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
    ports:
      - "8081:80"
```

``` jenkins
[MvcTest] $ /bin/bash /tmp/jenkins3923627102865574973.sh
---------------Remove-Orphans------------------

Removing network dotnetcoreapp1_default
------------------Config-----------------------

services:
  ci-build:
    command: /bin/bash -c "dotnet restore ./DotNetCoreApp.sln && dotnet publish ./DotNetCoreApp.sln
      -c Release -o ./obj/Docker/publish"
    image: microsoft/aspnetcore-build:1.0-2.0
    volumes:
    - /var/jenkins_home/workspace/MvcTest:/src:rw
    working_dir: /src
version: '3.0'

------------------Build------------------------

Creating network "dotnetcoreapp1_default" with the default driver

Creating dotnetcoreapp1_ci-build_1 ... 

[1A[2K
Creating dotnetcoreapp1_ci-build_1 ... [32mdone[0m
[1BAttaching to dotnetcoreapp1_ci-build_1
[36mci-build_1  |[0m   Restoring packages for /src/DotNetCoreApp/DotNetCoreApp.csproj...

[36mci-build_1  |[0m   Generating MSBuild file /src/DotNetCoreApp/obj/DotNetCoreApp.csproj.nuget.g.props.

[36mci-build_1  |[0m   Generating MSBuild file /src/DotNetCoreApp/obj/DotNetCoreApp.csproj.nuget.g.targets.
[36mci-build_1  |[0m   Restore completed in 30.06 sec for /src/DotNetCoreApp/DotNetCoreApp.csproj.
[36mci-build_1  |[0m   Restoring packages for /src/DotNetCoreApp/DotNetCoreApp.csproj...

[36mci-build_1  |[0m   Restore completed in 2.64 sec for /src/DotNetCoreApp/DotNetCoreApp.csproj.

[36mci-build_1  |[0m Microsoft (R) Build Engine version 15.6.84.34536 for .NET Core
[36mci-build_1  |[0m Copyright (C) Microsoft Corporation. All rights reserved.
[36mci-build_1  |[0m 

[36mci-build_1  |[0m   Restore completed in 97.25 ms for /src/DotNetCoreApp/DotNetCoreApp.csproj.
[36mci-build_1  |[0m   Restore completed in 24.37 ms for /src/DotNetCoreApp/DotNetCoreApp.csproj.

[36mci-build_1  |[0m   DotNetCoreApp -> /src/DotNetCoreApp/bin/Release/netcoreapp2.0/DotNetCoreApp.dll

[36mci-build_1  |[0m   DotNetCoreApp -> /src/DotNetCoreApp/obj/Docker/publish/

[36mdotnetcoreapp1_ci-build_1 exited with code 0

[0m---------------Publishing...------------------

Building dotnetcoreapp

Step 1/6 : FROM microsoft/aspnetcore:2.0
 ---> c4ca78cf9dca
Step 2/6 : ARG source
 ---> Using cache
 ---> 5ccef217b449
Step 3/6 : WORKDIR /app
 ---> Using cache
 ---> 21ca58736a23
Step 4/6 : EXPOSE 80
 ---> Using cache
 ---> fa9898960c84
Step 5/6 : COPY ${source:-obj/Docker/publish} .

 ---> a7d0c6386eb4
Step 6/6 : ENTRYPOINT ["dotnet", "DotNetCoreApp.dll"]
 ---> Running in dd4cfb6f5cb8

Removing intermediate container dd4cfb6f5cb8
 ---> d8dea36554b4
Successfully built d8dea36554b4
Successfully tagged dotnetcoreapp:latest
Recreating dotnetcoreapp2_dotnetcoreapp_1 ... 

[1A[2K
Recreating dotnetcoreapp2_dotnetcoreapp_1 ... [32mdone[0m
[1B---------------Clear-Images...------------------

Deleted: sha256:238c39ae8c13a1d6816361946429024cb46e6911329dad2d714033d2578da618
Deleted: sha256:12dfe61eae3ee68775237340f3e50e19554a653357baec0de52a50d0cff96652
Deleted: sha256:3f6beb8ffd03ee5d3b9fd0e9e5b46074d270cdb707fa07056087194a53be1dd4
---------------Clear-Containers...------------------

dotnetcoreapp1_ci-build_1
Finished: SUCCESS
```

最后通过浏览器访问`http://localhost:8081`

参考：

[http://www.cnblogs.com/stulzq/p/8627824.html](http://www.cnblogs.com/stulzq/p/8627824.html)
[https://github.com/muyinchen/woker/blob/master/%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E6%90%AD%E5%BB%BAJenkins%2BGithub%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83.md](https://github.com/muyinchen/woker/blob/master/%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95%E7%8E%AF%E5%A2%83%E6%90%AD%E5%BB%BA/%E6%89%8B%E6%8A%8A%E6%89%8B%E6%95%99%E4%BD%A0%E6%90%AD%E5%BB%BAJenkins%2BGithub%E6%8C%81%E7%BB%AD%E9%9B%86%E6%88%90%E7%8E%AF%E5%A2%83.md)

[http://www.cnblogs.com/LongJiangXie/p/7517909.html](http://www.cnblogs.com/LongJiangXie/p/7517909.html)

[http://www.cnblogs.com/JacZhu/p/6814848.html](http://www.cnblogs.com/JacZhu/p/6814848.html)
