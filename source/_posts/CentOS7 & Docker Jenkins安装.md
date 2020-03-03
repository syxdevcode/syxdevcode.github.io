---
title: CentOS7 & Docker Jenkins安装
date: 2017-06-02 10:20:16
tags: 
- Linux
- Jenkins
- CentOS7
- Docker
categories: 
- Jenkins
---
# CentOS7 jenkins配置.Net Core持续部署

## jenkins安装

[Jenkins官网](https://jenkins.io/download/ "jenkins官网")

### 1.创建Dockerfile

``` docker
# 使用touch创建空文件
sudo touch Dockerfile
sudo vim Dockerfile
```

### 2.填充一下内容

注：提前更新Docker镜像源

``` docker
FROM jenkins

USER root
#更新源并安装缺少的包
RUN apt-get update && apt-get install -y libltdl7 && apt-get update

# 使 jenkins 运行 docker 不需要 sudo
RUN groupadd -o -g 999 docker && usermod -aG docker jenkins
ARG dockerGid=999

RUN echo "docker:x:${dockerGid}:jenkins" >> /etc/group

# 解决Docker时区问题
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

USER jenkins
# 解决时区问题
ENV JAVA_OPTS -Duser.timezone=Asia/Shanghai

USER root
# 安装 docker-compose 因为等下构建环境的需要
RUN curl -L https://github.com/docker/compose/releases/download/1.21.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose

RUN chmod +x /usr/local/bin/docker-compose
```

注：可能出现

``` docker
[root@localhost Desktop]# curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
curl: (35) Peer reports incompatible or unsupported protocol version.
```

需要运行以下命令更新：

``` linux
yum update nss curl nss-util nspr
```

等待时间可能有点长，请耐心等待

运行结果：

``` docker
[root@localhost Desktop]# docker build . -t auto-jenkins
Sending build context to Docker daemon   2.56kB
Step 1/8 : FROM jenkins
 ---> 7b210b6c238a
Step 2/8 : USER root
 ---> Using cache
 ---> 9ce15bde1415
Step 3/8 : RUN apt-get update && apt-get install -y libltdl7 && apt-get update
 ---> Using cache
 ---> 3d428282a736
Step 4/8 : RUN groupadd -o -g 999 docker && usermod -aG docker jenkins
 ---> Using cache
 ---> 94f6440d5fae
Step 5/8 : ARG dockerGid=999
 ---> Using cache
 ---> 3cce8ba121fe
Step 6/8 : RUN echo "docker:x:${dockerGid}:jenkins" >> /etc/group
 ---> Using cache
 ---> a0f4fdd6cf3f
Step 7/8 : RUN curl -L https://github.com/docker/compose/releases/download/1.21.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
 ---> Running in fcdb1deaf0bf
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   617    0   617    0     0    226      0 --:--:--  0:00:02 --:--:--   226
100 10.3M  100 10.3M    0     0   591k      0  0:00:17  0:00:17 --:--:-- 1622k
Removing intermediate container fcdb1deaf0bf
 ---> 1909e02c39da
Step 8/8 : RUN chmod +x /usr/local/bin/docker-compose
 ---> Running in 409184886de5
Removing intermediate container 409184886de5
 ---> b0f1c2c53301
Successfully built b0f1c2c53301
Successfully tagged auto-jenkins:latest
```

出现以上 Successfully 内容代表安装Jenkins成功

## 启动Jenkins

需要先创建一个Jenkins的配置目录，并且挂载到docker 里的Jenkins目录下

``` linux
mkdir -p /var/jenkins_home
```

运行

``` docker
docker run --name jenkins -p 8080:8080 -p 50000:50000 \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -v $(which docker):/bin/docker \
    -v /var/jenkins_home:/var/jenkins_home \
    -d auto-jenkins
```

第一行构建一个名称为jenkins的容器,需要使用的 8080 跟 50000 端口,-p 是容器运行开放端口

第二行将宿主机上的 docker.sock 挂载到容器中的相应位置，使得容器中的 docker cli 能跟宿主机的 docker 通信

第三行将宿主机上面的 docker 命令行工具挂载到容器中，使 jenkins 用户能够执行 docker 命令

第四行挂载我们之前创建的配置文件存放目录到 jenkins 用户的 home（对的，jenkins 用户的 home 目录在 /var 下面）,建立宿主机的配置目录，挂载进docker容器里，这样容器里的Jenkins配置目录文件就会映射出来

第五行：使用auto-jenkins Image 并且后台启动

运行完这条指令后，出现一串很长的字符串以后，我们的jenkins已经成功启动，可以访问本机的 8080 端口来登录 Jenkins

通过命令`docker ps`查看运行的镜像

``` docker
[root@localhost Desktop]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                              NAMES
0c63f3ce79b5        auto-jenkins        "/bin/tini -- /usr/l…"   12 seconds ago      Up 5 seconds        0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   jenkins
```

删除docker

``` docker
docker rm -f jenkins
```

## 配置jenkins

进入容器内

``` docker
docker exec -it jenkins /bin/bash
```

查看密码：

``` docker
cat /var/jenkins_home/secrets/initialAdminPassword
```

``` linux
oot@localhost Desktop]# docker exec -it jenkins /bin/bash
root@0c63f3ce79b5:/# cat /var/jenkins_home/secrets/initialAdminPassword
fc648ef222a54ac690bec031834885f2
root@0c63f3ce79b5:/# exit
exit
```

复制输出的内容，粘贴到Administrator password，输入 exit 退出容器，此时进行下一步你会看到此界面，点击 Install suggested plugins

重启：

``` docker
docker restart jenkins
```

最后，安装推荐插件

忘记管理员密码

``` linux
[root@localhost Desktop]# cd /var/jenkins_home/users/admin/
[root@localhost admin]# ls
config.xml
[root@localhost admin]# vim config.xml
[root@localhost admin]# docker restart jenkins
jenkins
[root@localhost admin]#
```

把`<passwordHash>`节点的内容（图中黑色的那一串）换成

`#jbcrypt:$2a$10$DdaWzN64JgUtLdvxWIflcuQu2fgrrMSAMabF5TSrGK5nXitqK9ZMS`

重启，默认密码为：111111

参考网址：

[http://www.cnblogs.com/LongJiangXie/p/7517909.html](http://www.cnblogs.com/LongJiangXie/p/7517909.html)

[http://www.cnblogs.com/stulzq/p/8627360.html](http://www.cnblogs.com/stulzq/p/8627360.html)

[http://www.cnblogs.com/JacZhu/p/6814848.html](http://www.cnblogs.com/JacZhu/p/6814848.html)