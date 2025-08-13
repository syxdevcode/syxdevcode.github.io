---
title: CentOS7查看端口占用
date: 2017-11-30 08:43:46
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- CentOS7
---
# CentOS7查看端口占用

## Linux命令

netstat -nap #会列出所有正在使用的端口及关联的进程/应用

lsof -i :portnumber #portnumber要用具体的端口号代替，可以直接列出该端口听使用进程/应用

### 查看监听(Listen)的端口

netstat -lntp

### 检查端口被哪个进程占用

netstat -lnp|grep 88   #88请换为你的apache需要的端口，如：80

### 查看进程的详细信息

ps 1777

### 杀掉进程，重新启动

kill -9 1777        #杀掉编号为1777的进程（请根据实际情况输入）

service httpd start #启动apache

### 开启远程访问端口

  > 开启端口：

````linux
firewall-cmd --zone=public --add-port=6379/tcp --permanent
````

命令含义：

````linux
--zone #作用域

--add-port=27017/tcp #添加端口，格式为：端口/通讯协议

--permanent #永久生效，没有此参数重启后失效
````

> 重启防火墙

````linux
firewall-cmd --reload  #重启防火墙
````
