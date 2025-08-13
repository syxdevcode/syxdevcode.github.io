---
title: Linux安装JDK-CentOS7
date: 2020-08-31 09:48:13
tags:
- Linux
- CentOS7
- Java
- 环境变量
categories: 
- Java
---

## 下载安装包

在官网下载jdk 安装包：

[https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)

## 解压

```sh
# 解压 并输出到 /usr/local
tar -zxvf jdk-8u91-linux-x64.tar.gz -C /usr/local
```

## 配置

编辑 `/etc/profile`

`vim /etc/profile`

```sh
JAVA_HOME=/usr/local/jdk1.8.0_91
PATH=$PATH:$JAVA_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib
export JAVA_HOME PATH CLASSPATH
```

使配置生效：

`source /etc/profile`

## 验证

```sh
java
java -version
javac
```