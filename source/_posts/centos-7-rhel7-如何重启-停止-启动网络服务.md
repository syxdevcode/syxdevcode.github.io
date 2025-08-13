---
title: 'centos7/rhel7: 如何重启/停止/启动网络服务'
date: 2017-05-15 16:01:19
tags:
- Linux
- CentOS7
- Linux基础命令
categories:
- CentOS7
---
# centos7/rhel7: 如何重启/停止/启动网络服务

## 重启网络服务

```` linux
systemctl retart network.service
或
systemctl restart network
````

## 启动网络服务

```` linux
systemctl start network.service
或
systemctl start network
````

## 停止网络服务

```` linux
systemctl stop network.service
或
systemctl stop network
````

参考：[http://www.osetc.com/archives/1191.html](http://www.osetc.com/archives/1191.html "centos7/rhel7: 如何重启/停止/启动网络服务")