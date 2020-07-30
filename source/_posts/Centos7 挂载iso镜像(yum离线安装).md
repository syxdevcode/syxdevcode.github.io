---
title: Centos7 挂载iso镜像(yum离线安装)
date: 2020-05-11 16:57:45
tags:
- Linux
- CentOS7
- 安装部署
categories: 
- CentOS7
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

# (可选) 一个一个文件备份
mv CentOS-Base.repo CentOS-Base.repo.bak
cp CentOS-Media.repo CentOS-Media.repo.bak

mkdir bak
# 拷贝目录下所有.repo和.bak文件到 bak文件夹下
mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/bak
mv /etc/yum.repos.d/*.bak /etc/yum.repos.d/bak

# 新建 local.repo
touch local.repo
vi local.repo

# local.repo 内容
[local]
name=local_yum
baseurl=file:///media/cdrom
enable=1
gpgcheck=0
```

```sh
[local]　　　　　　　　　　　　　　 #库名称
name=local　　               #名称描述
baseurl=file:///media/cdrom   #yum源目录，源地址为rpm的目录
gpgcheck=1　　　　　　　　　　   #检查GPG-KEY，0为不检查，1为检查
enabled=1　　　　　　　　　　　  #是否用该yum源，0为禁用，1为使用
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6　　#gpgcheck=0时无需配置
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

[centos--软件源--本地软件源---离线安装](https://www.cnblogs.com/hl-piglet/p/8445988.html)

[CentOS 本地ISO 挂载并配置本地软件源](https://www.cnblogs.com/oftenlin/p/4325023.html)