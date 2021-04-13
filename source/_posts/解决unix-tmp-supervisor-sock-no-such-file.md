---
title: 解决unix-tmp-supervisor-sock-no-such-file
date: 2020-05-19 10:30:08
tags:
- Linux
- CentOS7
- 安装部署
- supervisor
categories: 
- supervisor
---

## 原因

```sh
unix:///tmp/supervisor.sock no such file
```

supervisor 默认配置会把 socket 文件和 pid 守护进程生成在/tmp/目录下，/tmp/目录是缓存目录，Linux 会根据不同情况自动删除其下面的文件。

## 修改配置

```sh
vi /etc/supervisord.conf

# 英文分号后的内容为注释
[unix_http_server]
;file=/tmp/supervisor.sock   ; (the path to the socket file)
file=/var/run/supervisor.sock   ; 修改为 /var/run 目录，避免被系统删除

[supervisord]
;logfile=/tmp/supervisord.log ; (main log file;default $CWD/supervisord.log)
logfile=/var/log/supervisord.log ; 修改为 /var/log 目录，避免被系统删除
pidfile=/var/run/supervisord.pid ; 修改为 /var/run 目录，避免被系统删除
...

[supervisorctl]
; 必须和'unix_http_server'里面的设定匹配
;serverurl=unix:///tmp/supervisor.sock ; use a unix:// URL  for a unix socket
serverurl=unix:///var/run/supervisor.sock ; 修改为 /var/run 目录，避免被系统删除
```

可选：

* 修改权限

```sh
sudo chmod 777 /run
sudo chmod 777 /var/log
```

* 创建supervisor.sock

```sh
sudo touch /var/run/supervisor.sock
sudo chmod 777 /var/run/supervisor.sock
```

## 更新配置文件

```sh
supervisorctl update
```

参考：

["unix:///tmp/supervisor.sock no such file" 错误处理 (亲测)](https://hacpai.com/article/1546398597198)

[https://blog.csdn.net/qq_28885149/article/details/79364685](解决unix:///tmp/supervisor.sock no such file的问题)
