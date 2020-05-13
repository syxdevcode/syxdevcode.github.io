---
title: CentOS7防火墙设置
date: 2020-05-13 17:58:13
tags:
- Linux
- CentOS7
categories: 
- CentOS7
---

## 常用命令

```sh
## 启动服务：
systemctl start firewalld.service

## 关闭服务：
systemctl stop firewalld.service

##重启服务：
systemctl restart firewalld.service

##显示服务的状态：
systemctl status firewalld.service

##开机自动启动：
systemctl enable firewalld.service

##禁用开机自动启动：
systemctl disable firewalld.service

##查看版本： 
firewall-cmd --version

##查看帮助： 
firewall-cmd --help

##显示状态： 
firewall-cmd --state

##查看所有打开的端口： 
firewall-cmd --zone=public --list-ports

## 开端口/服务
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-service=https --permanent
firewall-cmd --zone=public --add-port=5001/tcp --permanent

##更新防火墙规则： 
firewall-cmd --reload

##查看区域信息:  
firewall-cmd --get-active-zones

##查看指定接口所属区域： 
firewall-cmd --get-zone-of-interface=eth0

##拒绝所有包：
## 注意：通过SSH等远程连接，千万不要用，除非可以连接宿主机
--firewall-cmd --panic-on

##取消拒绝状态： 
firewall-cmd --panic-off

##查看是否拒绝： 
firewall-cmd --query-panic
```