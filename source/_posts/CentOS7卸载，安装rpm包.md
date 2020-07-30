---
title: CentOS7卸载，安装rpm包
date: 2020-07-30 13:29:41
tags:
- Linux
- CentOS7
- 安装部署
categories: 
- CentOS7
---

```sh
# 强制卸载：
rpm -e –-nodeps xxxxxx.rpm

# 强制安装：
rpm -ivh –-nodeps xxxxxx.rpm
```

示例：

```sh
rpm -e --nodeps glib2

# 进入挂载的镜像内
cd /media/cdrom/
cd Packages/
ls | grep glib2

[root@host Packages]# ls | grep glib2
compat-libpackagekit-glib2-16-0.8.9-1.el7.x86_64.rpm
glib2-2.54.2-2.el7.x86_64.rpm
glib2-devel-2.54.2-2.el7.x86_64.rpm
pulseaudio-libs-glib2-10.0-5.el7.x86_64.rpm

[root@host Packages]# rpm -ivh glib2-2.54.2-2.el7.x86_64.rpm 
```
