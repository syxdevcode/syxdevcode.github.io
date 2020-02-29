---
title: Docker容器无法访问宿主机-No route to host
date: 2018-06-29 09:05:05
tags:
- Docker
categories: 
- Docker
---
# docker容器无法访问宿主机-No route to host

## 方法一：关闭防火墙

centos关闭防火墙的操作为

```bash
systemctl stop firewalld
```

## 方法二： 在防火墙上开发指定端口

```bash
firewall-cmd --zone=public --add-port=2181/tcp --permanent
firewall-cmd --reload
```

参考：

[docker容器无法访问宿主机-No route to host](https://www.jianshu.com/p/96aebba5d3cc)

[浅谈Docker Bridge网络模式](https://wenfeng-gao.github.io/2016/05/20/docker-bridge-network.html)

[Docker 网络之理解 bridge 驱动](http://www.cnblogs.com/sparkdev/p/9217310.html)

[Docker 网络之进阶篇](https://www.cnblogs.com/sparkdev/p/9198109.html)