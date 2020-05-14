---
title: CentOS7离线安装supervisor
date: 2020-05-12 09:15:29
tags:
- Linux
- CentOS7
- Ubuntu
- 安装部署
categories: 
- CentOS7
---

## 安装 setuptools

下载[setuptools 46.2.0](https://pypi.org/project/setuptools/#files)离线包

参考：[installing](http://supervisord.org/installing.html)

```sh
## 解压
uzip -o -d setuptools-46.2.0 setuptools-46.2.0.zip
# 把 setuptools-46.2.0.zip 文件解压到 setuptools-46.2.0
# -o ：不提示的情况下覆盖文件
# -d：将文件解压缩到指定目录下
cd setuptools-46.2.0

## 安装
python setup.py install
```

需要安装 uzip 命令：

```sh
yum install zip
yum install unzip
```

## 从pip下载supervisor

下载[supervisor 4.2.0](https://pypi.org/project/supervisor/#files)离线包

参考：[installing](http://supervisord.org/installing.html)

进入文件所在目录：

```sh
## 解压
tar -zxvf supervisor-4.2.0.tar.gz
cd supervisor-4.2.0

## 安装
python setup.py install
```

验证安装是否成功

```sh
supervisorctl --help
```

## 配置supervisor


```sh
##创建supervisor所需目录
mkdir /etc/supervisord.d/

##创建supervisor配置文件
echo_supervisord_conf > /etc/supervisord.conf

##编辑supervisord.conf文件
vim /etc/supervisord.conf
```

在 `[include]` 标签中添加 `/etc/supervisord.d/*.conf`

```sh
[include]
files = relative/directory/*.ini  /etc/supervisord.d/*.conf
```

## 测试supervisor

### 常用命令

```sh
supervisord -c /etc/supervisord.conf ## 启动
supervisorctl shutdown  ## 关闭
supervisord -c /etc/supervisord.conf  ## 通过配置文件启动supervisor
supervisorctl -c /etc/supervisord.conf status  ## 查看状态
supervisorctl -c /etc/supervisord.conf reload  ## 重新载入配置文件 
supervisorctl -c /etc/supervisord.conf start [all]|[x]  ## 启动所有/指定的程序进程 
supervisorctl -c /etc/supervisord.conf stop [all]|[x]  ## 关闭所有/指定的程序进程
```

### 部署测试站点

新建 `WebApplication1.conf`:

```sh
touch /etc/supervisord.d/WebApplication1.conf
vim /etc/supervisord.d/WebApplication1.conf
```

内容：

注意：supervisor配置文件中的注释使用英文分号 `;`

```sh
[program:WebApplication1]
command=dotnet WebApplication1.dll
directory=/ldjc/webapi
environment=ASPNETCORE__ENVIRONMENT=Production ;,ASPNETCORE_URLS="http://0.0.0.0:5100"
stopsignal=INT
stderr_logfile=/var/log/WebApplication1.err.log
stdout_logfile=/var/log/WebApplication1.out.log
```

详细命令参考：[Configuration File](http://supervisord.org/configuration.html#file-format)

查看站点日志：

```sh
vim /var/log/WebApplication1.err.log
vim /var/log/WebApplication1.out.log
```

## 开启防火墙

注意：针对各种云，因为有专门策略，所以不需要操作防火墙；

```sh
firewall-cmd --zone=public --add-port=5000/tcp --permanent
firewall-cmd --zone=public --add-port=5001/tcp --permanent
firewall-cmd --reload

## 重新查询端口是否开放
firewall-cmd --query-port=5000/tcp

## 查看监听的端口
netstat -lnpt

#关闭端口
firewall-cmd --zone=public --remove-port=5000/tcp --permanent  

## 查看防火墙所有开放的端口
firewall-cmd --zone=public --list-ports
```

## 测试站点

```sh
curl -X GET "http://localhost:5000/weatherforecast" -H "accept: application/json"
curl -X GET "https://127.0.0.1:5001/weatherforecast" -H "accept: application/json"
curl -X GET "http://192.125.30.82:5000/weatherforecast" -H "accept: application/json"
```

## 开机启动

脚本:[centos-systemd-etcs](https://github.com/Supervisor/initscripts/blob/master/centos-systemd-etcs)

```Bash
[root@host supervisor-4.2.0]# touch /usr/lib/systemd/system/supervisord.service
[root@host supervisor-4.2.0]# vim /usr/lib/systemd/system/supervisord.service
[root@host supervisor-4.2.0]# systemctl enable supervisord
Created symlink from /etc/systemd/system/multi-user.target.wants/supervisord.service to /usr/lib/systemd/system/supervisord.service.
[root@host supervisor-4.2.0]# systemctl is-enabled supervisord
enabled
[root@host supervisor-4.2.0]# 
```

supervisord.service:

```sh
# supervisord service for systemd (CentOS 7.0+)
# by ET-CS (https://github.com/ET-CS)
[Unit]
Description=Supervisor daemon

[Service]
Type=forking
ExecStart=/usr/bin/supervisord
ExecStop=/usr/bin/supervisorctl $OPTIONS shutdown
ExecReload=/usr/bin/supervisorctl $OPTIONS reload
KillMode=process
Restart=on-failure
RestartSec=42s

[Install]
WantedBy=multi-user.target
```


设置成功后，可以使用下列命令：

```sh
# 重启指定应用
supervisorctl restart <application name>

# 停止指定应用
supervisorctl stop <application name>

# 启动指定应用
supervisorctl start <application name>

# 重启所有应用
supervisorctl restart all

# 停止所有应用
supervisorctl stop all

# 启动所有应用
supervisorctl start all
```

参考：

[Installing](http://supervisord.org/installing.html)

[Supervisor离线安装、管理](https://www.cnblogs.com/sailq21/p/9227592.html)