---
title: Docker 推送镜像
date: 2022-09-07 18:22:41
tags:
  - Docker
categories:
  - Docker
---

## 自定义 Docker Hub 服务器 Http 支持

```shell
vim /etc/docker/daemon.json
```

添加 hub 地址：

```json
{
  "insecure-registries": ["10.10.0.105:8080"]
}
```

示例：

```json
{
  "registry-mirrors": ["https://lhao27k5.mirror.aliyuncs.com"],
  "insecure-registries" : ["10.10.0.105:8080"]
}
```

重启docker 服务

```sh
systemctl restart docker
```

## 登录 Docker Hub

```shell
docker login 10.10.0.105:8080
```

## 镜像生成，上传

```shell
# 生成镜像
docker build -t 10.10.0.105:8080/myhub/lims:0.0.1 /lims/git/Admin.NET

# 推送 -a 推送所有镜像
docker push 10.10.0.105:8080/myhub/lims:0.0.1
```
