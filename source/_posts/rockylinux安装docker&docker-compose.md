---
title: Rockylinux安装docker&docker-compose
date: 2022-09-19 09:44:07
tags:
  - Docker Compose
  - Docker
  - RockyLinux
  - Linux
categories:
  - Docker
---

## 查看Rockylinux版本

```sh
# 查看系统版本
cat /etc/rocky-release

## 查看内核版本
cat /proc/version
# 或
uname -r
```
<!--more-->

## 安装docker

```sh
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf -y install docker-ce docker-ce-cli containerd.io docker-compose-plugin
sudo systemctl --now enable docker

# 查看版本
docker --version
```

## 安装docker Compose

运行下列命令安装最新稳定的 Docker Compose 文件：

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/v2.11.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

如果有更新版本，只需要将上述命令中的 v2.11.0 替换为最新的版本号即可。请不要忘记数字前的 “v” 。

最后，使用下列命令赋予二进制文件可执行权限：

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

运行下列命令检查安装的 Docker Compose 版本：

```sh
docker-compose version
Docker Compose version v2.11.0
```

参考：

[https://docs.rockylinux.org/gemstones/docker/](https://docs.rockylinux.org/gemstones/docker/)