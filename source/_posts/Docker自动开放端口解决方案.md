---
title: Docker自动开放端口解决方案
date: 2022-09-08 14:49:15
tags:
  - Ubuntu
  - Docker
  - Docker Compose
  - Linux基础
  - Linux
  - ufw
  - iptables
  - 安全策略
categories:
  - 防火墙
---

## Docker 自动开放端口解决方案

### 简单暴力法不让 docker 使用 iptables

修改 docker 的配置文件`/etc/docker/daemon.json`(若没有就新建一个), 添加如下内容:

```json
{
  "iptables": false
}
```

重启 docker:

```shell
systemctl restart docker
```

再启动 docker 容器, 就不会修改防火墙了.

## 直接 iptables 移除并修改

## 使用 DOCKER-USER 方案

[https://holywhite.com/archives/489](https://holywhite.com/archives/489)

## 使用 DFW 方案

[https://github.com/pitkley/dfw](https://github.com/pitkley/dfw)

## 修改绑定为 127.0.0.1

```yml
version: "3"
services:
  lims-gateway:
    image: lims-gateway:0.0.1
    restart: always
    container_name: lims-gateway
    ports:
      - "127.0.0.1::80"
    networks:
      - net0
    volumes:
      - /etc/localtime:/etc/localtime
    env_file:
      - ./var.env
networks:
  net0:
```

参考：

[关于 docker 自动开放端口解决方案
](https://www.jianshu.com/p/c41668271d1f)
