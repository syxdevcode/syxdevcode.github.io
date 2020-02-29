---
title: (AWS)Ubuntu 16.04.4 LTS 安装Nginx
date: 2018-04-25 14:53:17
tags:
- Linux
- Ubuntu
- AWS
- Nginx
- 安装部署
categories:
- Nginx
---
# (AWS)Ubuntu 16.04.4 LTS 安装Nginx

## 安装nginx

[nginx官网](http://nginx.org/en/linux_packages.html "官网")

``` linux
sudo apt-get install nginx
```

>Ubuntu安装之后的文件结构大致为：

* 所有的配置文件都在/etc/nginx下，并且每个虚拟主机已经安排在了/etc/nginx/sites-available下
* 程序文件在/usr/sbin/nginx
* 日志放在了/var/log/nginx中
* 并已经在/etc/init.d/下创建了启动脚本nginx
* 默认的虚拟主机的目录设置在了/var/www/nginx-default (有的版本 默认的虚拟主机的目录设置在了/var/www, 请参考/etc/nginx/sites-available里的配置)

## 启动nginx

``` linux
sudo /etc/init.d/nginx start
```

或者

``` linux
sudo service nginx start
```

## 停止

``` linux
sudo service nginx stop
sudo service nginx restart
```

## 检查config配置问题

``` nginx
sudo nginx -t -c /etc/nginx/nginx.conf
```

然后就可以访问了，`http://localhost/`，默认80端口