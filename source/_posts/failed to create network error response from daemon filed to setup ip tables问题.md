---
title: failed to create network error response from daemon filed to setup ip tables问题
date: 2022-09-09 14:26:38
tags:
  - Linux
  - CentOS7
  - Ubuntu
  - iptables
  - Docker
  - Docker Compose
categories:
  - Docker Compose
---

容器在运行中，调整 iptables 策略，停止防火墙，在操作容器就会报这个错误，我们可以重启 docker 解决此问题。

```shell
# 重启docker
systemctl restart docker.service

# 启动
docker compose up -d
```
