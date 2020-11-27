---
title: TomCat Url忽略大小写
date: 2020-11-26 15:21:54
tags:
- Linux
- CentOS7
- Java
- Tomcat
categories: 
- Tomcat
---

注：tomcat 8不生效

## 修改context.xml

`vim /usr/local/apache-tomcat-8.5.40/conf/context.xml`

```sh
<Context caseSensitive="false">

    <!-- Default set of monitored resources. If one of these changes, the    -->
    <!-- web application will be reloaded.                                   -->
    <WatchedResource>WEB-INF/web.xml</WatchedResource>
    <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>

    <!-- Uncomment this to disable session persistence across Tomcat restarts -->
    <!--
    <Manager pathname="" />
    -->
</Context>
```

## 重启

```sh
systemctl status tomcat8
systemctl restart tomcat8
```

## 验证

```sh
# 查看监听的端口
# Tomcat默认8080端口
netstat -lnpt
```
