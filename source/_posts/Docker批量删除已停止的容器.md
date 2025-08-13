---
title: Docker批量删除已停止的容器
date: 2022-09-30 15:51:27
tags:
  - Ubuntu
  - Docker
  - Linux
categories:
  - Docker
---

## 方法一

```sh
#显示所有的容器，过滤出Exited状态的容器，取出这些容器的ID，
sudo docker ps -a|grep Exited|awk '{print $1}'

#查询所有的容器，过滤出Exited状态的容器，列出容器ID，删除这些容器
sudo docker rm `docker ps -a|grep Exited|awk '{print $1}'`
```

## 方法二

```sh
#删除所有未运行的容器（已经运行的删除不了，未运行的就一起被删除了）
sudo docker rm $(sudo docker ps -a -q)
```

## 方法三

```sh
#根据容器的状态，删除Exited状态的容器
sudo docker rm $(sudo docker ps -qf status=exited)
```

## 方法四

```sh
#Docker 1.13版本以后，可以使用 docker containers prune 命令，删除孤立的容器。
sudo docker container prune
```

```sh
#删除所有镜像
sudo docker rmi $(docker images -q)
```

参考：

[如何批量删除Docker中已停止的容器？-可以有多种方式](https://blog.csdn.net/CSDN_duomaomao/article/details/78587103)