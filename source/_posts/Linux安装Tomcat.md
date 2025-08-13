---
title: Linux安装Tomcat
date: 2020-08-31 13:45:44
tags:
- Linux
- CentOS7
- Java
- Tomcat
categories: 
- Tomcat
---

## 下载

[https://tomcat.apache.org/](https://tomcat.apache.org/)

## 安装

```sh
# 解压的 /usr/local
tar -zxvf apache-tomcat-8.5.40.tar.gz -C /usr/local

# 进入 目录
cd /usr/local/apache-tomcat-8.5.40/bin

# 启动
./startup.sh

# 停止
./shutdown.sh
```

## 验证

```sh
# 查看监听的端口
# Tomcat默认8080端口
netstat -lnpt
```