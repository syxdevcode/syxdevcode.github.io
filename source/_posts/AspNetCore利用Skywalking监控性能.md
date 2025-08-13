---
title: AspNetCore利用Skywalking监控性能
date: 2018-06-28 16:40:42
tags:
- Elasticsearch
- Docker
- DotNetCore
categories: 
- Skywalking
---
# AspNetCore利用Skywalking监控性能

[SkyWalking](https://github.com/apache/incubator-skywalking/blob/master/README_ZH.md)开源项目由吴晟于2015年创建，同年10月在GitHub上作为个人项目开源。

SkyWalking项目的核心目标，是针对微服务、Cloud Native、容器化架构，提供应用性能监控（APM）和分布式调用链追踪能力。

2017年11月，SkyWalking社区正式决定，寻求加入Apache基金会，希望能使项目成为更为开放、全球化和强大的APM开源产品，并加强来自社区的合作和交流。最终实现构建一款功能强大、简单易用的开源APM产品。

2017年12月8日，Apache软件基金会孵化器项目管理委员会 ASF IPMC宣布“SkyWalking全票通过，进入Apache孵化器”。

** 软件环境**

```txt
CentOS7
Docker 18.03.1-ce
ElasticSearch5.X
AspDotNetCore2.x
skywalking 5.0.0-beta
```

## Docker安装ElasticSearch5.X

新建docker-compose.yml文件：

```bash
touch docker-compose.yml
```

编辑，填入以下内容：

```docker
version: '2.2'
services:
  elasticsearch5.6:
    image: docker.elastic.co/elasticsearch/elasticsearch:5.6.10
    container_name: elasticsearch5.6
    environment:
      - cluster.name=CollectorDBCluster
      - xpack.security.enabled=false
      - network.host= 0.0.0.0
      - thread_pool.bulk.queue_size=1000
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata1:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - esnet
volumes:
  esdata1:
    driver: local
networks:
  esnet:
```

注：需要使用xpack.security.enabled=false，关闭xpack身份验证。

启动：

```docker
docker-compose up -d
docker ps # 查看容器
```

## 部署skywalking

官网：[https://github.com/apache/incubator-skywalking/releases](https://github.com/apache/incubator-skywalking/releases)

参考：[https://github.com/apache/incubator-skywalking/blob/master/docs/en/Deploy-backend-in-standalone-mode.md](https://github.com/apache/incubator-skywalking/blob/master/docs/en/Deploy-backend-in-standalone-mode.md)

下载[5.0.0-beta](https://github.com/apache/incubator-skywalking/archive/v5.0.0-beta.tar.gz)

解压：

```bash
tar -xzvf apache-skywalking-apm-incubating-5.0.0-beta.tar.gz
```

** 修改配置**

（1） 修改/bin目录下webappService.sh

--collector.ribbon.listOfServers=192.168.0.110:10800

修改localhost为本地IP地址：

localhost=>192.168.0.110

（2） 修改config目录下application.yml

修改localhost为本地IP地址：

localhost=>192.168.0.110

进入解压后的目录，运行：

```bash
./bin/startup.sh
```

结果：

```bash
SkyWalking Collector started successfully!
SkyWalking Web Application started successfully!
```

查看主机监控的端口：

```bash
netstat -ntlp
```

结果：

```bash
tcp6       0      0 192.168.0.110:12800     :::*                    LISTEN      4147/java
tcp6       0      0 192.168.0.110:10800     :::*                    LISTEN      4147/java
tcp6       0      0 :::8080                 :::*                    LISTEN      4153/java
tcp6       0      0 :::9200                 :::*                    LISTEN      3682/docker-proxy
tcp6       0      0 :::8083                 :::*                    LISTEN      3971/docker-proxy
tcp6       0      0 :::8084                 :::*                    LISTEN      3868/docker-proxy
tcp6       0      0 :::9300                 :::*                    LISTEN      3672/docker-proxy
tcp6       0      0 192.168.0.110:11800     :::*                    LISTEN      4147/java
```

其中8080，10800，11800，12800为skywalking程序使用的端口，
8083，8084为AspDotNetCore使用端口
9200，9300为EalsticSearch使用端口

## 运行dotnetcore应用

### 1.获取AspDotNetCore项目

```bash
git clone https://github.com/syxdevcode/SkyWalking.Sample.git ## 克隆项目
git pull origin master ## 更新代码（可选）
```

### 2.使用docker-compose命令启动

```bash
docker-compose up -d --build
```

注：运行可能遇到问题：

```bash
Unable to load the service index for source https://api.nuget.org/v3/index.json
```

可能因为镜像服务问题：

解决：

```bash
vim /etc/docker/daemon.json
```

修改前：

```text
{
  "registry-mirrors": ["https://lhao27k5.mirror.aliyuncs.com"]
}
```

修改后：

```text
{
  "registry-mirrors": ["https://registry.docker-cn.com"]
}
```

重启docker服务：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```

之后继续执行docker-compose命令，同时，不要忘记将elasticsearch容器启动。

查看容器IP：

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' skywalkingsample_skywalkingfrontend_1
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' skywalkingsample_skywalkingbackend_1
```

### 3.firewalld添加端口

在docker容器内部无法访问宿主机-No route to host问题，需要在宿主机防火墙添加端口，或者关闭防火墙（不推荐）

(1) 关闭/启动防火墙：

```bash
systemctl start firewalld
systemctl stop firewalld
```

(2) 添加端口

```bash
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --zone=public --add-port=11800/tcp --permanent
firewall-cmd --reload
```

### 4.测试AspDotNetCore程序接口

（1） Backend项目Put接口

``` bash
curl --header "Content-Type: application/json" \
  --request PUT \
  --data '{"Id":"1","Name":"Test1"}' \
  http://192.168.0.110:8084/api/apps
```

(2) Backend项目Get接口

```bash
curl http://localhost:8084/api/apps
[{"id":1,"name":"Test1"}]
```

### 5.查看skywalking

访问：[http://localhost:8080/#/monitor/dashboard](http://localhost:8080/#/monitor/dashboard)

![QQ截图20180629105145.png](/img/QQ截图20180629105145.png)

参考：

[https://github.com/apache/incubator-skywalking/blob/master/docs/en/Deploy-backend-in-standalone-mode.md](https://github.com/apache/incubator-skywalking/blob/master/docs/en/Deploy-backend-in-standalone-mode.md)

[利用Skywalking-netcore监控你的应用性能](https://www.jianshu.com/p/3ddd986c7581?from=timeline&isappinstalled=0)

[Apache SkyWalking 为.NET Core带来开箱即用的分布式追踪和应用性能监控](https://www.cnblogs.com/liuhaoyang/p/skywalking-dotnet-v02-release.html)