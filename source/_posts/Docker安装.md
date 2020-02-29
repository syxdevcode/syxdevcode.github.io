---
title: Docker安装
date: 2018-04-27 14:40:52
tags:
- Linux
- Docker
- 安装部署
categories: 
- Docker
---
# Docker安装

[官网](https://docs.docker.com/install/linux/docker-ce/centos/#uninstall-old-versions "官网")

## 安装Docker CE

``` linux
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

$ sudo yum install docker-ce
```

启动

``` linux
sudo systemctl start docker
```

验证是否成功

``` linux
sudo docker run hello-world
```

## 配置镜像加速

### 安装／升级你的Docker客户端

推荐安装1.10.0以上版本的Docker客户端

### 如何配置镜像加速器

针对Docker客户端版本大于1.10.0的用户

您可以通过修改daemon配置文件`/etc/docker/daemon.json`来使用加速器：

``` linux
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://lhao27k5.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

或者使用`ttps://registry.docker-cn.com`

## 卸载

### 1.卸载Docker包

``` linux
sudo yum remove docker-ce
```

### 2.主机上的mages, containers, volumes或自定义配置文件不会自动删除。 删除所有mages, containers, volumes：

``` linux
sudo rm -rf /var/lib/docker
```

您必须手动删除任何定义配置文件。
