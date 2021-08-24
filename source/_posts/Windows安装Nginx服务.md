---
title: Windows安装Nginx服务
date: 2021-08-24 15:22:37
tags:
- Windows
- Nginx
- Windows服务
- 安装部署
categories:
- Nginx
---

Nginx/Win32是运行在一个控制台程序，而非windows服务方式的。

Nginx/Win32可以使用以下开关来管理它：

```sh
Nginx -s stop   #快速关闭Nginx，可能不保存相关信息，并迅速终止web服务。（快速退出）
Nginx -s quit    #平稳关闭Nginx，保存相关信息，有安排的结束web服务。（平滑退出）
Nginx -s reload   #因改变了Nginx相关配置，需要重新加载配置而重载。（重新加载配置)
Nginx -s reopen  #重新打开日志文件。(重新加载日志)
```

<!--more-->
## 使用nssm安装

软件下载：[https://nssm.cc/release/nssm-2.24.zip](https://nssm.cc/release/nssm-2.24.zip)

下载完解压，cmd执行：

```sh
nssm install nginx
```

```ps1
PS C:\Users\test\Downloads\Compressed\nssm-2.24\win64> cmd
Microsoft Windows [版本 10.0.18363.1734]
(c) 2019 Microsoft Corporation。保留所有权利。

C:\Users\test\Downloads\Compressed\nssm-2.24\win64>nssm install nginx
Administrator access is needed to install a service.
```

![微信截图_20210824153602.png](/img/微信截图_20210824153602.png)

windows服务管理器停止Nginx服务后，访问浏览器仍然能看到网站。查看进程，你会发现，Nginx的其中一个进程还在运行！nginx进程(根据nginx.conf的配置worker_processes 1;)，Fork出来的进程显然没有被停止，结果就是nginx永远关不掉。因此彻底关闭nginx请使用taskkill命令！

那么我们只好做个`stop_nginx`脚本来处理nginx停止的所有操作：

```ps1
@echo off
net stop nginx
taskkill /F /IM nginx.exe>nul
```

参考：

[将nginx安装为Windows服务](https://zhuanlan.zhihu.com/p/265119336)

## 使用Windows Service Wrapper安装

项目地址：[https://github.com/kohsuke/winsw](https://github.com/kohsuke/winsw)

下载后把下载的文件放在 `Nginx` 安装目录，并修改名称为 `nginx-service.exe`，然后分别创建 `nginx-service.exe.config`，`nginx-service.xml`文件,把这两个文件放在Nginx安装目录下。

`nginx-service.exe.config` 内容如下:

```xml
<configuration>
  <startup>
    <supportedRuntime version="v2.0.50727" />
    <supportedRuntime version="v4.0" />
  </startup>
  <runtime>
    <generatePublisherEvidence enabled="false"/> 
  </runtime>
</configuration>
```

`nginx-service.xml` 内容如下（根据实际情况对路径进行调整）：

```xml
<service>
  <id>nginx-service</id>
  <name>Nginx Service</name>
  <description>High Performance Nginx Service</description>
  <logpath>C:\Program Files\Nginx\logs</logpath>
  <log mode="roll-by-size">
    <sizeThreshold>10240</sizeThreshold>
    <keepFiles>8</keepFiles>
  </log>
  <executable>C:\Program Files\Nginx\nginx.exe</executable>
  <startarguments>-p C:\Program Files\Nginx</startarguments>
  <stopexecutable>C:\Program Files\Nginx\nginx.exe</stopexecutable>
  <stoparguments>-p C:\Program Files\Nginx -s stop</stoparguments>
</service>
```

安装`nginx`服务 `nginx-service.exe install`
删除`nginx`服务 `sc delete Nginx Service`

参考：

[Windows中将nginx添加到服务](https://www.cnblogs.com/lujiulong/p/9025066.html)