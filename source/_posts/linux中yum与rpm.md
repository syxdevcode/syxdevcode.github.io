---
title: linux中yum与rpm
date: 2020-05-11 10:03:34
tags:
- Linux
- CentOS7
- Ubuntu
- 安装部署
categories: 
- Linux
---

## 源代码形式

1.绝大多数开源软件都是直接以原码形式发布的

2.源代码一般会被打成.tar.gz的归档压缩文件

3.源代码需要编译成为二进制形式之后才能够运行使用

4.源代码基本编译流程：

1）.configure 检查编译环境；

2）make对源代码进行编译；

3）make insall 将生成的可执行文件安装到当前计算机中

## RPM

1.源代码形式的特点：操作复杂、编译时间长、极易出现问题、依赖关系复杂

2.为了方便，RPM（redhat package manager）

3.RPM通过将代码基于特定平台系统编译为可执行文件，并保存依赖关系，来简化开源软件的安装管理。针对不同的系统设定不同的包

4.常用命令规范：linuxcast-1.2.0-30.el6.1686.rpm 包名-版本号-适用平台-32/64-rpm

5.使用

```bash
## 安装
rpm –i software.rpm

## 卸载
rpm -e software.rpm

## 升级形式安装
rpm –U software.rpm

## 例子
rpm –ivh http://www.linuxcast.net/software.rpm ## 支持通过http\ftp协议形式安装

-v 显示详细信息；-h显示进度条
查询功能：rpm –qa 
列出全部已经安装的.rpm软件  rpm –qa |grep ***
```

## YUM

1.rpm软件包形式的管理虽然方便，但是需要手工解决软件包的依赖关系。很多时候安装一个软件安装一个软件需要安装1个或者多个其他软件，手动解决时，很复杂，yum解决这些问题。Yum是rpm的前端程序，主要目的是设计用来自动解决rpm的依赖关系，其特点：

1）自动解决依赖关系；
2）可以对rpm进行分组，基于组进行安装操作；
3）引入仓库概念，支持多个仓库；
4）配置简单

2.yum仓库用来存放所有的现有的.rpm包，当使用yum安装一个rpm包时，需要依赖关系，会自动在仓库中查找依赖软件并安装。仓库可以是本地的，也可以是HTTP、FTP、nfs形式使用的集中地、统一的网络仓库。

3.仓库的配置文件/etc/yum.repos.d目录下

4.使用：

```sh
## 1 安装
yum install 全部安装
yum install package1 安装指定的安装包package1
yum groupinsall group1 安装程序组group1

## 2 更新和升级
yum update 全部更新
yum update package1 更新指定程序包package1
yum check-update 检查可更新的程序
yum upgrade package1 升级指定程序包package1
yum groupupdate group1 升级程序组group1

## 3 查找和显示
yum info package1 显示安装包信息package1
yum list 显示所有已经安装和可以安装的程序包
yum list package1 显示指定程序包安装情况package1
yum groupinfo group1 显示程序组group1信息yum search string 根据关键字string查找安装包

## 4 删除程序
yum remove &#124; erase package1 删除程序包package1
yum groupremove group1 删除程序组group1
yum deplist package1 查看程序package1依赖情况

## 5 清除缓存
yum clean packages 清除缓存目录下的软件包
yum clean headers 清除缓存目录下的 headers
yum clean oldheaders 清除缓存目录下旧的 headers
yum clean, yum clean all (= yum clean packages; yum clean oldheaders) 清除缓存目录下的软件包及旧的headers
```

5.查询软件：

可以使用 `yumsearch **`

参考：

[linux中yum与rpm区别](https://blog.csdn.net/ziyun_xiaoyan/article/details/54341823)

[linux yum命令详解](https://www.cnblogs.com/chuncn/archive/2010/10/17/1853915.html)
