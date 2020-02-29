---
title: Docker基础-镜像
date: 2018-05-09 09:00:50
tags:
- Linux
- Docker
- CentOS7
categories: 
- Docker
---

** 三大核心概念 **

镜像，容器，仓库

**镜像是 Docker 的三大核心概念之一**。Docker 运行容器前需要本地存在对应的镜像，如果本地没有对应的镜像，Docker 会尝试从默认的镜像仓库下载。当然用户也可以通过配置，使用自定义的镜像仓库。
本文将介绍镜像的具体操作，包括使用 pull 命令从 Docker Hub 的镜像仓库中拉取(下载)公共镜像；查看本地已有的镜像信息；使用 search 命令搜索镜像；删除镜像标签和镜像文件；创建用户自定义镜像并上传到 Docker Hub 镜像仓库。
与镜像相关的操作都被定义在 **docker image** 子命令中，虽然不带 image 的格式依然被兼容，但带上 image 后会让命令更容易理解，也会有更好的自动补全效果。

## 获取镜像

本地镜像是运行容器的前提，所以在运行容器前我们需要使用 docker image pull 命令从网络上的镜像仓库把镜像拉取到本地。该命令的格式为：

** docker image pull [OPTIONS] NAME[:TAG|@DIGEST] **

如果只指定了镜像的名称，默认会选择拉取 latest 标签标记的镜像。比如我们要拉取最新的 ubuntu 镜像：

``` docker
docker image pull ubuntu
```

![1](/img/QQ截图20180509141612.png)

该命令实际拉取的是 ubuntu:latest 镜像。从上图中可以看到，docker 的镜像其实被分成了很多的层，每层保存一些特定的文件。上面的命令实际相当于：

``` docker
docker image pull registry.hub.docker.com/ubuntu:latest
```

即从默认的数据仓库服务器 registry.hub.docker.com 中拉取 ubuntu 仓库中的最新镜像。如果我们感觉从 Docker Hub 上拉取镜像太慢，可选择从其它的数据仓库服务器上拉取，比如 Docker Hub 在国内部署的服务器：

``` docker
docker image pull registry.docker-cn.com/library/ubuntu:latest
```

镜像下载到本地后就可运行容器了，比如：

``` docker
docker run --rm ubuntu echo hello docker
```

--rm :退出时，删除容器

![1](/img/QQ截图20180509163105.png)

## 查看镜像信息

 `docker image ls` 或 `docker images` 命令可以列出本地存储的镜像
![2](/img/QQ截图20180509163307.png)

输出的信息中包含的内容有：

REPOSITORY：说明镜像来自哪个仓库，比如 ubuntu 或 registry.docker-cn.com/library/ubuntu。

TAG：镜像的标签信息，比如 14.04 或 latest。

IMAGE ID：标识镜像的 ID 号。

CREATED：创建镜像的时间。

SIZE：镜像大小。

其中镜像的 ID 信息十分重要，它唯一的标识了镜像。

TAG 信息用来标记来自同一个仓库(比如 ubuntu)的不同镜像。例如 ubuntu 仓库中有多个镜像，可以通过 TAG 信息来区

分它们，TAG 13.04、14.04 和 16.04 都代表了不同的发行版本。

使用 `docker image tag` 命令为本地的镜像添加新的标签还可以方便我们的使用，比如为 ubuntu:latest 镜像添加下面的标签：

``` docker
docker image tag ubuntu:latest oldubuntu
```

![3](/img/QQ截图20180509163748.png)

使用`docker image inspect`命令可以查看镜像详细信息

``` docker
docker image inspect ubuntu:latest
```

![4](/img/QQ截图20180509164159.png)

它输出的是一个 JSON 格式的信息，一般情况下我们会有的放矢的通过 -f 选项取其中的某一部分。比如只获取镜像的 Architecture 信息：

``` docker
docker image inspect -f {{".Architecture"}} ubuntu:latest
```

## 搜索镜像

除了直接在 Docker Hub 的官方网站上搜索镜像资源，还可以通 docker search 命令以命令行的方式进行搜索，比如搜索 mysql 镜像：

``` docker
docker search mysql
```

这个命令的价值有限，因为我们在选择镜像时还是需要慎重的。往往要在 Docker Hub 的官方网站上查看镜像相关的详细信息，然后才会决定是否使用，而 docker search 命令提供的信息过于有限。

## 删除镜像

 docker image rm 命令传递镜像的标签或 ID，这两种方式略微有些区别。

### 使用进行的标签删除镜像

对于被多个标签引用的镜像 ID，删除标签时只是把那个标签删掉了，并不会真正删除镜像文件。

### 使用镜像 ID 删除镜像

 docker 检测到该镜像 ID 被引用了多次就机智的报错了，并且终止了删除操作。同样如果由其它的镜像引用了该 ID 的镜像， docker 同样会报错并终止删除操作。所以，只有当一个镜像不被多个标签引用，也没其它镜像引用它时，才可以被通过镜像 ID 删除。

## 创建镜像

我们可以通过不同的方式创建镜像，比如基于已有容器进行创建和基于 Dockerfile 文件进行创建。

这里只介绍如何通过 `docker container commit` 命令基于已有容器创建镜像

我们先启动一个以 ubuntu:latest 为镜像的容器，然后在当前目录下创建一个名为 testfile 的文件:

``` docker
docker run -it ubuntu:latest bash
```

``` docker
[root@localhost Desktop]# docker run -it ubuntu:latest bash
root@2a45b0b39bf8:/# touch testfile
root@2a45b0b39bf8:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  testfile  usr
boot  etc  lib   media  opt  root  sbin  sys  tmp       var
root@2a45b0b39bf8:/#
```

在文件创建后退出容器，但要记住该容器的 ID 为： 2a45b0b39bf8 。然后执行下面的命令创建镜像：

``` docker
docker container commit -m "add file testfile." 2a45b0b39bf8 testimage
```

镜像创建成功后，你可以在镜像列表中看到名称为 testimage 的镜像

```docker
docker images
```

下面运行一个基于 testimage 的容器，看看 nickfile 是否存在。--rm：退出时自动删除

```docker
docker run --rm testimage ls
```

在容器中创建的文件 testfile 已经被成功的添加到 testimage 镜像中了。

## 导入导出镜像

当碰到没有网络的环境时，如何获取镜像呢？答案是在能够获得镜像的环境中把镜像导出为文件，然后通过 U 盘等设备拷贝到目标环境中，再进行导入。

### 导出镜像

通过 `docker image save` 命令可以把镜像导出为本地文件，比如导出 ubuntu:latest 镜像为 ubuntu1804.tar：

```docker
docker image save -o ubuntu1804.tar ubuntu:latest
```

一般我们还会再压缩一下，这样最终的文件会小不少：

```docker
tar -czf ubuntu1804.tar.gz ubuntu1804.tar
```

![4](/img/QQ截图20180509172150.png)

### 导入镜像

把 ubuntu1804.tar.gz 文件拷贝到目标系统上后先要解压出 ubuntu1804.tar 文件：

``` linux
tar -xf ubuntu1804.tar.gz
```

然后通过 docker image load 命令执行镜像的导入操作：

``` linux
docker image load -i ubuntu1804.tar
```

这样就 OK 了，用 docker image ls 命令看看，是不是已经可以看到 ubuntu:latest 镜像了！

## 上传镜像

可以使用 **`docker image push`** 命令把镜像上传到镜像仓库服务器，默认是上传到 Docker Hub 的镜像仓库，此时事先需要注册用户并进行登录。上传镜像的命令格式为：

``` docker
docker image push NAME[:TAG]
```

在 Docker Hub 注册账号，并通过 docker login 命令完成了登录操作(需要输入用户名和密码进行验证)。接下来就可把本地的镜像上传到镜像仓库服务器了。在上传前需要给镜像打上合法的标签(用户账号/仓库名称:TAG)，比如：

``` linux
[root@localhost Desktop]# docker image tag --help

Usage: docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]

Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Options:
```

``` docker
docker image tag azcli:1.0 ljfpower/azcli:latest
```

最后上传这个标签就行了：

``` docker
docker image push ljfpower/azcli:latest
```

上传后你就可以在 Docker Hub 上看到这个镜像了。

参考：

[https://www.cnblogs.com/sparkdev/p/8901728.html](https://www.cnblogs.com/sparkdev/p/8901728.html)