---
title: Docker容器进入的方式
date: 2018-05-03 16:37:21
tags:
- Linux
- CentOS7
- Docker
categories: 
- Docker
---
# Docker容器进入的方式

## 1.使用docker exec进入Docker容器

docker在1.3.X版本之后还提供了一个新的命令exec用于进入容器，这种方式相对更简单一些，下面我们来看一下该命令的使用：

``` docker
sudo docker exec --help
```

``` linux
[root@localhost toor]# sudo docker exec --help

Usage: docker exec [OPTIONS] CONTAINER COMMAND [ARG...]

Run a command in a running container

Options:
  -d, --detach               Detached mode: run command in the background
      --detach-keys string   Override the key sequence for detaching a
                             container
  -e, --env list             Set environment variables
  -i, --interactive          Keep STDIN open even if not attached
      --privileged           Give extended privileges to the command
  -t, --tty                  Allocate a pseudo-TTY
  -u, --user string          Username or UID (format:
                             <name|uid>[:<group|gid>])
  -w, --workdir string       Working directory inside the container
[root@localhost toor]#
```

``` docker
[root@localhost toor]# docker exec -it a46fa8852e07 /bin/bash
root@a46fa8852e07:/# exit
exit
[root@localhost toor]#
```

参考：

[https://www.cnblogs.com/xhyan/p/6593075.html](https://www.cnblogs.com/xhyan/p/6593075.html)