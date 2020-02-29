---
title: Docker Dockerfile详解
date: 2018-04-27 15:01:22
tags:
- Docker
categories: 
- Docker
---
# Docker Dockerfile详解

[官网](https://docs.docker.com/engine/reference/builder/#dockerfile-examples)

## Docker简介

Docker项目提供了构建在Linux内核功能之上，协同在一起的的高级工具。其目标是帮助开发和运维人员更容易地跨系统跨主机交付应用程序和他们的依赖。Docker通过Docker容器，一个安全的，基于轻量级容器的环境，来实现这个目标。这些容器由镜像创建，而镜像可以通过命令行手工创建或 者通过Dockerfile自动创建。

## Dockerfile简介

Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像。它们简化了从头到尾的流程并极大的简化了部署工作。Dockerfile从FROM命令开始，紧接着跟随者各种方法，命令和参数。其产出为一个新的可以用于创建容器的镜像。

## 基本结构

一般而言，Dockerfile 的内容分为四个部分：基础镜像信息、维护者信息、镜像操作指令和容器启动时执行指令。

``` nginx
# This dockerfile uses the Ubuntu image
# VERSION 2
# Author: docker_user
# Command format: Instruction [arguments / command] …

# 第一行必须指定基于的容器镜像
FROM ubuntu

# 维护者信息
MAINTAINER docker_user docker_user@email.com

# 镜像的操作指令
RUN echo “deb http://archive.ubuntu.com/ubuntu/ raring main universe” >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo “\ndaemon off;” >> /etc/nginx/nginx.conf

# 容器启动时执行指令
CMD /usr/sbin/nginx
```

## Dockerfile 命令

### FROM

FROM命令可能是最重要的Dockerfile命令。该命令定义了使用哪个基础镜像启动构建流程。基础镜像可以为任意镜像。如果基础镜像没有被发现，Docker将试图从Docker image index来查找该镜像。FROM命令必须是Dockerfile的首个命令。

``` docker
# Usage: FROM [image name]
FROM ubuntu
```

### MAINTAINER

声明作者;这个命令放在Dockerfile的起始部分，虽然理论上它可以放置于Dockerfile的任意位置。这个命令用于声明作者，并应该放在FROM的后面。

注意：MAINTAINER 指令已经被抛弃，建议使用 LABEL 指令。

``` docker
# Usage: MAINTAINER [name]
MAINTAINER authors_name
```

### LABEL

格式为:

``` docker
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

LABEL 指令为镜像添加标签。一个 LABEL 就是一个键值对。

下面是一些例子:

```docker
LABEL "com.example.vendor"="ACME Incorporated"
LABEL com.example.label-with-value="foo"
LABEL version="1.0"
LABEL description="This text illustrates \that label-values can span multiple lines."
```

我们可以给镜像添加多个 LABEL，需要注意的是，每条 LABEL 指令都会生成一个新的层。所以最好是把添加的多个 LABEL 合并为一条命令：

```docker
LABEL multi.label1="value1" multi.label2="value2" other="value3"
```

也可以写成这样：

``` docker
LABEL multi.label1="value1" \
      multi.label2="value2" \
      other="value3"
```

如果新添加的 LABEL 和已有的 LABEL 同名，则新值会覆盖掉旧值。
我们可以使用 docker inspect 命令查看镜像的 LABEL 信息。

### RUN

``` docker
有两种格式，分别为：

RUN <command>

RUN [“executable”, “param1”, “param2”]。
```

RUN命令是Dockerfile执行命令的核心部分。它接受命令作为参数并用于创建镜像。不像CMD命令，RUN命令用于创建镜像（在之前commit的层之上形成新的层）。
前者将在 shell 终端中运行命令，即 /bin/sh -c，后者则使用 exec 执行。指定使用其他终端可以通过第二种方式实现，例如 RUN  [“/bin/bash”, “-c”, “echo hello”]。
每条 RUN 指令将在当前镜像的基础上执行指定命令，并提交为新的镜像。当命令较长时可以使用 \ 来换行。

``` docker
# Usage: RUN [command]
RUN aptitude install -y riak
```

### CMD

支持三种格式：

``` docker
CMD [“executable”, “param1”, “param2”] 使用 exec 执行，推荐方式。

CMD command param1 param2 在 /bin/sh 中执行，提供给需要交互的应用。

CMD [“param1”, “param2”] 提供给 ENTRYPOINT 的默认参数。
```

指定启动容器时执行的命令，每个 Dockerfile 只能有一条 CMD 命令。如果指定了多条 CMD 命令，只有最后一条会被执行。如果用户在启动容器时指定了要运行的命令，则会覆盖掉 CMD 指定的命令。

和RUN命令相似，CMD可以用于执行特定的命令。和RUN不同的是，这些命令不是在镜像构建的过程中执行的，而是在用镜像构建容器后被调用。

``` docker
# Usage 1: CMD application "argument", "argument", ..
CMD "echo" "Hello docker!"
```

### EXPOSE

格式为：

``` docker
EXPOSE <port> [<port>…]
```

告诉 Docker 服务，容器需要暴露的端口号，供互联系统使用。在启动容器时需要通过 -P 参数让 Docker 主机分配一个端口转发到指定的端口。使用 -p 参数则可以具体指定主机上哪个端口映射过来。

``` docker
# Usage: EXPOSE [port]
EXPOSE 8080
```

### ENV

ENV命令用于设置环境变量。这些变量以”key=value”的形式存在，并可以在容器内被脚本或者程序调用。这个机制给在容器中运行应用带来了极大的便利。

``` docker
# Usage: ENV key value
ENV SERVER_WORKS 4
```

### ADD

格式为：

``` docker
ADD <src> <dest>
```

ADD命令有两个参数，源和目标。它的基本作用是从源系统的文件系统上复制文件到目标容器的文件系统。如果源是一个URL，那该URL的内容将被下载并复制到容器中。其中 `<src>` 可以是 Dockerfile 所在目录的一个相对路径(文件或目录)；也可以是一个 URL；还可以是一个 tar 文件(自动解压为目录)。

``` docker
# Usage: ADD [source directory or URL] [destination directory]
ADD /my_app_folder /my_app_folder
```

### COPY

格式：

``` docker
COPY <src> <dest>
```

复制本地主机的 `<src>` (为 Dockerfile 所在目录的相对路径，文件或目录) 为容器中的 `<dest>`。目标路径不存在时，会自动创建。当使用本地目录为源目录时，推荐使用 COPY。

### ENTRYPOINT

有两种格式：

``` docker
ENTRYPOINT [“executable”, “param1”, “param2”]

ENTRYPOINT command param1 param2 (shell 中执行)
```

配置容器启动后执行的命令，并且不可被 docker run 提供的参数覆盖。

每个 Dockerfile 中只能有一个 ENTRYPOINT，当指定多个时，只有最后一个起效。

ENTRYPOINT 帮助你配置一个容器使之可执行化，如果你结合CMD命令和ENTRYPOINT命令，你可以从CMD命令中移除“application”而仅仅保留参数，参数将传递给ENTRYPOINT命令。

``` docker
# Usage: ENTRYPOINT application "argument", "argument", ..
# Remember: arguments are optional. They can be provided by CMD
# or during the creation of a container.
ENTRYPOINT echo
# Usage example with CMD:
# Arguments set with CMD can be overridden during *run*
CMD "Hello docker!"
ENTRYPOINT echo
```

### VOLUME

格式为：

``` docker
VOLUME ["/data"]
```

也可以使用 VOLUME 指令添加多个数据卷：

``` docker
VOLUME ["/data1", "/data2"]
```

创建一个可以从本地或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等。
VOLUME命令用于让你的容器访问宿主机上的目录。

``` docker
# Usage: VOLUME ["/dir_1", "/dir_2" ..]
VOLUME ["/my_files"]
```

### USER

指定运行容器时的用户名或 UID，后续的 RUN 也会使用指定用户。当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如：RUN groupadd -r postgres && useradd -r -g postgres postgres。

``` docker
# Usage: USER [UID]
USER 751
```

### WORKDIR

格式为：

``` docker
WORKDIR /path/to/workdir
```

为后续的 RUN、CMD、ENTRYPOINT 指令配置工作目录。可以使用多个 WORKDIR 指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。例如：

``` docker
# Usage: WORKDIR /path
WORKDIR ~/
```

### ONBUILD

格式为：

``` docker
ONBUILD [INSTRUCTION]
```

配置当所创建的镜像作为其他新创建镜像的基础镜像时，所执行的操作指令。例如，Dockerfile 使用如下的内容创建了镜像 image-A。

``` docker
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build –dir /app/src
```

如果基于 image-A 创建新的镜像时，新的 Dockerfile 中使用 FROM image-A 指定基础镜像时，会自动执行 ONBUILD 指令内容，等价于在后面添加了两条指令。

``` docker
FROM image-A
#automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build –dir /app/src
```

### 如何使用Dockerfiles

编写完成 Dockerfile 之后，可以通过 docker build 命令来创建镜像。

使用Dockerfiles和手工使用Docker Daemon运行命令一样简单。脚本运行后输出为新的镜像ID。

``` docker
# Build an image using the Dockerfile at current location
# Example: sudo docker build -t [name] .
sudo docker build -t my_mongodb .
```

基本的格式为 docker build [选项] 路径，该命令将读取指定路径下(包括子目录)的 Dockerfile ，并将该路径下所有内容发送给 docker 服务端，由服务端来创建镜像。因此一般建议放置 Dockerfile 的目录为空目录。

另外，可以通过 .dockerignore 文件来让 docker 忽略路径下的目录和文件。 要指定镜像的标签信息，可以通过 -t 选项来实现。
例如，指定 Dockerfile 所在路径为 /tmp/docker_builder/，并且希望生成镜像标签为 build_repo/first_image，可以使用下面的命令：

``` docker
sudo docker build -t build_repo/first_image /tmp/docker_builder/
```

参考：

[http://www.cnblogs.com/sparkdev/p/6357614.html](http://www.cnblogs.com/sparkdev/p/6357614.html)

[https://yeasy.gitbooks.io/docker_practice/content/introduction/what.html](https://yeasy.gitbooks.io/docker_practice/content/introduction/what.html)

[https://www.cnblogs.com/boshen-hzb/p/6400272.html](https://www.cnblogs.com/boshen-hzb/p/6400272.html "参考")
