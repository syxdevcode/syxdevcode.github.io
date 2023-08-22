---
title: Docker-Compose安装部署Jenkins
date: 2023-08-16 10:28:50
tags:
  - Ubuntu
  - Jenkins
  - Docker Compose
  - Docker
categories:
  - Jenkins
---

## 简介

Jenkins是一个开源的、提供友好操作界面的持续集成(CI)工具，起源于Hudson（Hudson是商用的），主要用于持续、自动的构建/测试软件项目、监控外部任务的运行。Jenkins用Java语言编写，可在Tomcat等流行的servlet容器中运行，也可独立运行。通常与版本管理工具(SCM)、构建工具结合使用。常用的版本控制工具有SVN、GIT，构建工具有Maven、Ant、Gradle。

<!--more-->
## 安装

创建文件夹：`mkdir -p /opt/jenkins/jenkins_home`

新建docker-compose配置文件：

`vim /opt/jenkins/compose.yml`

```yml
version: '3'

networks:
  default:
    external:
      name: docker_compose_net

services: 
  jenkins:
    restart: unless-stopped
    image: jenkins/jenkins:2.419
    container_name: jenkins
    ports:
      - 8082:8080  # 访问Jenkins服务端口
      - 50000:50000
    volumes:
      - /etc/localtime:/etc/localtime # 时区
      - /opt/jenkins/jenkins_home:/var/jenkins_home  # jenkins_home目录
      - /var/run/docker.sock:/var/run/docker.sock
      #- /usr/bin/docker:/usr/bin/docker                # 容器内使用docker命令
      #- /usr/local/bin/docker-compose:/usr/local/bin/docker-compose    
    environment:
      TZ : Asia/Shanghai # 时区配置亚洲上海
      JAVA_OPTS : "-server -Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8 -Xms1024m -Xmx1024m -XX:PermSize=256m -XX:MaxPermSize=512m" # 处理内存占用过高问题
```

`-Dfile.encoding=UTF-8 -Dsun.jnu.encoding=UTF-8` ：处理中文乱码问题

如果您在其他机器上设置了一个或多个基于JNLP的Jenkins代理程序，而这些代理程序又与 jenkinsci/blueocean 容器交互（充当“主”Jenkins服务器，或者简称为“Jenkins主”）， 则这是必需的。默认情况下，基于JNLP的Jenkins代理通过TCP端口50000与Jenkins主站进行通信。 您可以通过“ 配置全局安全性” 页面更改Jenkins主服务器上的端口号。如果您要将您的Jenkins主机的JNLP代理端口的TCP端口 值更改为51000（例如），那么您需要重新运行Jenkins（通过此 docker run …​命令）并指定此“发布”选项 -p 52000:51000，其中最后一个值与Jenkins master上的这个更改值相匹配，第一个值是Jenkins主机的主机上的端口号， 通过它，基于JNLP的Jenkins代理与Jenkins主机进行通信 - 例如52000。

注意：10080和10022端口可能会被现代浏览器阻止访问。

**名称解释:**

JNLP：JNLP（Java Network Launching Protocol ）是java提供的一种可以通过浏览器直接执行java应用程序的途径，它使你可以直接通过一个网页上的url连接打开一个java应用程序。
Java桌面应用程序以JNLP的方式发布，如果版本升级后，不需要再向所有用户发布版本，只需要更新服务器的版本，这就相当于让java应用程序有了web应用的优点。

## 启动&移除

```sh
# 启动
docker-compose -f /opt/jenkins/compose.yml up -d

# 移除
docker-compose -f /opt/jenkins/compose.yml down -v

# 更新docker-compose
docker-compose -f /opt/jenkins/compose.yml up -d --build

# 第一次访问新的Jenkins实例时，系统会要求您使用自动生成的密码对其进行解锁
# 查看 密码
docker logs jenkins
```

## 配置从节点

* 1，设置工作目录，手动创建根目录。
注意：远程工作目录必填。
![Snipaste_2023-08-19_16-30-41.png](/img1/Snipaste_2023-08-19_16-30-41.png)

* 2，设置git目录
![Snipaste_2023-08-19_16-31-09.png](/img1/Snipaste_2023-08-19_16-31-09.png)

保存之后，会跳转到显示连接到主节点的命令行页面。

注意：加上 "-Dfile.encoding=UTF-8" "-Dsun.jnu.encoding=UTF-8" 处理中文乱码问题，例如：

`java "-Dfile.encoding=UTF-8" "-Dsun.jnu.encoding=UTF-8" -jar agent.jar -jnlpUrl ......`。

![Snipaste_2023-08-19_16-38-20.png](/img1/Snipaste_2023-08-19_16-38-20.png)

* 3，在从节点新建bat批处理程序，把相关操作系统的命令复制进去即可。

![Snipaste_2023-08-19_16-39-51.png](/img1/Snipaste_2023-08-19_16-39-51.png)

## 设置

### 设置国内源

```txt
Dashboard => Manage Jenkins => Plugin Manager => Advanced

Update Site 设置成  https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
```

## 安装插件

进入 系统管理=>插件管理，安装插件即可。

![Snipaste_2023-08-19_16-43-10.png](/img1/Snipaste_2023-08-19_16-43-10.png)

安装常用的插件：Git,GitBlit,Nodejs,PowerShell。

nodejs版本管理：

设置nodejs镜像地址：`https://npm.taobao.org/mirrors/node/`。

![https://npm.taobao.org/mirrors/node/](/img1/https://npm.taobao.org/mirrors/node/)

## 中文乱码

1，**power shell 首行加上**

`$OutputEncoding = [System.Text.Encoding]::UTF8`

2，**登录到Jenkins控制台。**

* 导航到 系统管理 -> 系统配置。
* 在 `Global properties` 下的 `Environment variables` 部分，添加一个新的环境变量 `JAVA_TOOL_OPTIONS`，并将值设置为 `-Dfile.encoding=UTF8`。
* 保存配置并重启Jenkins服务。

或者在Windows系统的环境变量加上 `JAVA_TOOL_OPTIONS=-Dfile.encoding=UTF8`

![Snipaste_2023-08-19_17-20-18.png](/img1/Snipaste_2023-08-19_17-20-18.png)

3，**设置系统的默认编码为UTF-8**：

* 打开控制面板并选择“时钟和区域”。
* 点击“区域”选项卡。
* 在“管理”部分，点击“更改系统区域设置”。
* 在“管理区域设置”窗口中，点击“更改系统区域设置”。
* 将“当前系统区域设置”和“当前用户区域设置”都设置为UTF-8。
* 重新启动计算机让更改生效。

![Snipaste_2023-08-19_17-22-23.png](/img1/Snipaste_2023-08-19_17-22-23.png)

参考：

[IIS应用程序池标识（程序池账户）ApplicationPoolIdentify](https://www.cnblogs.com/cplemom/p/11247577.html)

[jenkins windows节点中文乱码问题解决](https://blog.csdn.net/chenjxj123/article/details/128218242)