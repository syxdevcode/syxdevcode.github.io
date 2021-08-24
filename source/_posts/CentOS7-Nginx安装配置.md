---
title: CentOS7-Nginx安装配置
date: 2017-05-18 09:46:54
tags:
- Linux
- CentOS7
- Nginx
- 安装部署
categories:
- Nginx
---
## Nginx安装

### Yum安装方式

````sh
[root@localhost /]# yum install epel-release
[root@localhost /]# yum install nginx
[root@localhost /]# systemctl start nginx #启动
[root@localhost /]# systemctl enable nginx #可用
Created symlink from /etc/systemd/system/multi-user.target.wants/nginx.service to /usr/lib/systemd/system/nginx.service.
````

将nginx 设置为启动系统自动启动nginx

````sh
[root@localhost /]# echo "/usr/local/nginx/sbin/nginx" >> /etc/rc.local
````

### nginx.conf配置文件

```conf
[root@localhost /]# cd /etc/nginx #定位到nginx安装目录
[root@localhost nginx]# vim nginx.conf #通过vim打开nginx.conf配置文件进行配置
```

在http节点中，添加配置

```conf
#设定负载均衡的服务器列表
upstream load_balance_server {
    #weigth参数表示权值，权值越高被分配到的几率越大
    server 192.168.1.131:5100   weight=5;
    server 192.168.1.132:5100   weight=1;
    server 192.168.1.130:5100   weight=6;
}
```

在http->server节点中添加配置：

```conf
location / {
    root   html;
    index  index.aspx index.html index.htm;
    #其中load_balance_server 对应着upstream设置的集群名称
    proxy_pass         http://load_balance_server;
    #设置主机头和客户端真实地址，以便服务器获取客户端真实IP
    proxy_set_header   Host             $host;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
}
```

#### 报错

nginx 启动失败

```sh
[root@localhost nginx]# service nginx restart
Redirecting to /bin/systemctl restart  nginx.service
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
[root@localhost nginx]# systemctl status nginx -l
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Thu 2017-05-18 09:34:41 CST; 6s ago
  Process: 121270 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 121265 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 117382 (code=exited, status=0/SUCCESS)

May 18 09:34:41 localhost.localdomain systemd[1]: Starting The nginx HTTP and reverse proxy server...
May 18 09:34:41 localhost.localdomain nginx[121270]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
May 18 09:34:41 localhost.localdomain nginx[121270]: nginx: [emerg] bind() to 0.0.0.0:5100 failed (13: Permission denied)
May 18 09:34:41 localhost.localdomain nginx[121270]: nginx: configuration file /etc/nginx/nginx.conf test failed
May 18 09:34:41 localhost.localdomain systemd[1]: nginx.service: control process exited, code=exited status=1
May 18 09:34:41 localhost.localdomain systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
May 18 09:34:41 localhost.localdomain systemd[1]: Unit nginx.service entered failed state.
May 18 09:34:41 localhost.localdomain systemd[1]: nginx.service failed.
```

权限拒绝，经检查发现是开启selinux 导致的。 直接关闭

`getenforce`   这个命令可以查看当前是否开启了selinux 如果输出 disabled 或 permissive 那就是关闭了

如果输出 enforcing 那就是开启了 selinux

1、临时关闭selinux

```sh
setenforce 0    ##设置SELinux 成为permissive模式
setenforce 1    ##设置SELinux 成为enforcing模式
```

2、永久关闭selinux,

修改 `/etc/selinux/config` 文件

将 `SELINUX=enforcing` 改为 `SELINUX=disabled`

重启机器即可

### 重启nginx服务

```sh
service nginx restart
```

参考：

[http://www.cnblogs.com/Andon_liu/p/5353522.html](http://www.cnblogs.com/Andon_liu/p/5353522.html "Nginx安装")

[http://www.cnblogs.com/jingmoxukong/p/5945200.html](http://www.cnblogs.com/jingmoxukong/p/5945200.html "Nginx详细配置")