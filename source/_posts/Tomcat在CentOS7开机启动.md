---
title: Tomcat在CentOS7开机启动
date: 2020-09-17 11:06:30
tags:
- Linux
- CentOS7
- Java
- Tomcat
categories: 
- Tomcat
---

## Tomcat配置为系统服务

创建Tomcat8服务文件

```sh
touch /usr/lib/systemd/system/tomcat8.service
vim /usr/lib/systemd/system/tomcat8.service
```

tomcat8.service文件内容：

```sh
[Unit]
Description=Tomcat8 
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]  
Type=forking

Environment="JAVA_HOME=/usr/local/jdk1.8.0_91"
ExecStart=/usr/local/apache-tomcat-8.5.40/bin/startup.sh
ExecReload=/usr/local/apache-tomcat-8.5.40/bin/startup.sh
ExecStop=/usr/local/apache-tomcat-8.5.40/bin/shutdown.sh

[Install]
WantedBy=multi-user.target
```

```sh
systemctl enable tomcat8.service
systemctl is-enabled tomcat8
systemctl daemon-reload
systemctl restart tomcat8
```

## 验证

```sh
# 查看监听的端口
# Tomcat默认8080端口
netstat -lnpt
```

参考：

[centos7配置tomcat开机自启动](https://blog.csdn.net/qq_43080036/article/details/90064320)

[CentOS 7下Tomcat 安装与配置（Tomcat开机启动）](https://cloud.tencent.com/developer/article/1333879)