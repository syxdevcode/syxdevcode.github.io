---
title: Docker禁止自动开放端口
date: 2023-05-22 11:01:47
tags:
  - Ubuntu
  - Linux基础
  - Linux
  - iptables
  - Docker
  - Docker Compose
  - ufw
  - 安全策略
categories:
  - 防火墙
---

## 查看网络

```sh
iptables -L DOCKER
```

## 配置Docker

```sh
vim /etc/default/docker

#修改文件，此处设置等同于在创建容器时手动指定iptables=false参数
DOCKER_OPTS="--dns 8.8.8.8 --dns 8.8.4.4 -iptables=false"
```

```sh
vim /etc/docker/daemon.json
{
    "iptables": false
}
#此处对更改设置之前创建的容器也有效，编辑后需重启docker服务
```

### 重启docker服务

```sh
service docker start
service docker stop
service docker restart
```

<!--more-->
## 编辑 ufw 默认规则链

```sh
# ubuntu(ufw)操作为
vim /etc/default/ufw

DEFAULT_FORWARD_POLICY="ACCEPT"
```

net为当前网桥的名称，可通过ifconfig查看，10.10.13.0/24为网桥的子网段：

```sh
docker network inspect docker_compose_net | grep Subnet
"Subnet": "10.10.13.0/24",
```

```sh
# ubuntu(ufw):
vim /etc/ufw/before.rules

# 在 *filter 前面添加下面内容
*nat
:POSTROUTING ACCEPT [0:0]

-A POSTROUTING ! -o docker0 -s 10.10.13.0/24 -j MASQUERADE
-A POSTROUTING ! -o docker0 -s 10.10.14.0/24 -j MASQUERADE

# COMMIT 是文件最后一行，注意位置
COMMIT
```

重启一下 ufw 规则

```sh
sudo ufw disable
sudo ufw enable

# 重启ufw
sudo systemctl restart ufw
```

## 测试

```sh
sudo ufw deny 9200

sudo ufw allow 9200
```

## 问题处理

1，docker内容器无法访问其他容器的端口

```sh
# 开放docker内容器的访问权限
ufw allow from 172.17.0.1/24

# 刷新防火墙配置
ufw reload
```

2，使用ufw放开docker内的端口访问不生效

```sh
# 一般情况下docker外的端口想要开放端口访问直接执行以下指令即可
ufw allow 端口号

# 但是docker内的端口还需要先开放容器内的端口访问权限才行
ufw route allow 端口号
```

参考：

[无视系统防火墙的docker](https://www.binss.me/blog/docker-pass-through-system-firewall/)

[解决docker容器开启端口映射后，会自动在防火墙上打开端口的问题](https://www.cnblogs.com/qjfoidnh/p/11567309.html)

[Ubuntu防火墙ufw对Docker容器端口的策略问题处理](https://www.jianshu.com/p/ccc5a003fa7e)

[docker and ufw serious problems](https://github.com/moby/moby/issues/4737)

[关于docker自动开放端口解决方案](https://www.jianshu.com/p/c41668271d1f)