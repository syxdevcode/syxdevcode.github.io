---
title: Centos7 挂载iso镜像(yum离线安装)
date: 2020-05-11 16:57:45
tags:
- Linux
- CentOS7
- Ubuntu
- 安装部署
categories: 
- Linux
---

## 挂载iso文件到挂载点

```sh
## 创建文件夹
mkdir -p /media/cdrom

## 挂载iso文件到挂载点
mount -o loop /media/CentOS-7-x86_64-DVD-1804.iso /media/cdrom

##查看挂载状态
df -h

## 重新挂载系统分区
mount -a
```
## 修改yum的配置文件，使用本地ISO做yum源

```sh
cd /etc/yum.repos.d/
mv CentOS-Base.repo CentOS-Base.repo.bak
cp CentOS-Media.repo CentOS-Media.repo.bak
vi CentOS-Media.repo
```

```sh
[c7-media]
name=CentOS-$releasever - Media
baseurl=file:///media/CentOS/
        file:///media/cdrom/
        file:///media/cdrecorder/
gpgcheck=1
enabled=1 #启用yum
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

## 清除缓存

```sh
yum clean all
yum list
```

## 测试

```sh
yum install libicu
```

参考：

[CentOS 本地ISO 挂载并配置本地软件源](https://www.cnblogs.com/oftenlin/p/4325023.html)