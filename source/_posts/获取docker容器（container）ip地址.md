---
title: 获取docker容器（container）ip地址
date: 2018-05-16 14:40:11
tags:
- Docker
- AWS
- Ubuntu
categories: 
- Docker
---
# 获取docker容器（container）ip地址

## 使用命令方式

* 命令

```docker
sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-ID>
```

* 命令2

``` docker
sudo docker inspect <container id>

sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' <container_name_or_id>
```

## 进入容器内

命令：

```docker
sudo docker exec -it mynginx /bin/bash
cat /etc/hosts
```

```docker
ubuntu@ip-172-31-16-20:~/dotnet$ sudo docker exec -it mynginx /bin/bash
root@70e571d3bcab:/# cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.4      70e571d3bcab
```

## 在 ~/.bashrc 中写一个 bash 函数

```bash
sudo vim ~/.bashrc
```

添加如下内容：

```docker
function docker_ip() {
    sudo docker inspect --format '{{ .NetworkSettings.IPAddress }}' $1
}
```

然后执行：

```bash
source ~/.bashrc
```

最后调用函数：

``` bash
docker_ip mynginx
```

## 所有容器名称及其IP地址

```docker
sudo docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(docker ps -aq)
```

如果使用docker-compose命令将是:

```docker
docker inspect -f '{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```

```docker
sudo docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)
```

可能遇到的问题：

```docker
ubuntu@ip-172-31-16-20:~/dotnet$ sudo docker inspect -f '{{.Name}} - {{.NetworkSettings.IPAddress }}' $(docker ps -aq)
Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.37/containers/json?all=1: dial unix /var/run/docker.sock: connect: permission denied
"docker inspect" requires at least 1 argument.
See 'docker inspect --help'.

Usage:  docker inspect [OPTIONS] NAME|ID [NAME|ID...] [flags]

Return low-level information on Docker objects
```

执行以下命令,将当前用户添加 docker 组：

```bash
sudo usermod -a -G docker $USER
或
sudo usermod -aG docker $USER
```

注销，重新登录:

```bash
logout
```

注意：注销并重新登录是必需的，因为除非您的会话关闭，否则组更改不会生效。

参考：
[https://blog.csdn.net/sannerlittle/article/details/77063800](https://blog.csdn.net/sannerlittle/article/details/77063800)
