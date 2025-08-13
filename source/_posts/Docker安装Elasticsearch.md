---
title: Docker安装Elasticsearch
date: 2018-06-25 09:01:07
tags:
- 安装部署
- 分布式
- Elasticsearch
- Docker
categories: 
- Elasticsearch
---
# Docker安装Elasticsearch

## 用Dockeredit安装Elasticsearch

Elasticsearch也可用作Docker镜像。 镜像使用centos:7作为基本图像。

所有发布的Docker镜像和标签列表可以在[www.docker.elastic.co](www.docker.elastic.co)找到。 源代码可以在GitHub上找到。

X-Pack是一个Elastic Stack的扩展，可将安全性，警报，监控，报告和图形功能捆绑到一个易于安装的软件包中。 虽然X-Pack组件旨在无缝协同工作，您也可以轻松地启用或禁用要使用的功能。
在Elasticsearch 5.0.0之前，您必须安装单独的Shield，Watcher和Marvel插件，以获得X-Pack中捆绑在一起的功能。 使用X-Pack，您不再需要担心每个插件是否具有正确的版本，而只需安装您正在运行的Elasticsearch版本的X-Pack就可以了。

## 拉去镜像

镜像仓库：[https://www.docker.elastic.co](https://www.docker.elastic.co)

```docker
docker pull docker.elastic.co/elasticsearch/elasticsearch:6.3.0
docker pull docker.elastic.co/elasticsearch/elasticsearch-oss:6.3.0
```

## 运行Elasticsearch

### 开发模式

使用以下命令可以快速启动Elasticsearch以进行开发或测试：

```docker
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.0
```

### 生产模式

Linux

该vm.max_map_count设置应该在/etc/sysctl.conf中永久设置：

```docker
$ vim /etc/sysctl.conf
vm.max_map_count = 262144
```

使命令生效：sysctl -p

要在临时应用该设置： sysctl -w vm.max_map_count=262144

参考：[https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-prod-mode](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-prod-mode)

#### docker-compose启动

启动命令：

```docker
docker-compose up -d
```

节点elasticsearch侦听localhost:9200，而elasticsearch2 谈判elasticsearch在桥接网络。

这个例子还使用了 名为卷的Docker，它将被调用esdata1，esdata2如果尚不存在，将会创建它。

docker-compose.yml:

```docker
version: '2.2'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.3.0
    container_name: elasticsearch
    environment:
      - cluster.name=CollectorDBCluster
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
    networks:
      - esnet
  elasticsearch2:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.3.0
    container_name: elasticsearch2
    environment:
      - cluster.name=CollectorDBCluster
      - network.host= 0.0.0.0
      - thread_pool.bulk.queue_size=1000
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - "discovery.zen.ping.unicast.hosts=elasticsearch"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata2:/usr/share/elasticsearch/data
    networks:
      - esnet

volumes:
  esdata1:
    driver: local
  esdata2:
    driver: local

networks:
  esnet:
```

要停止群集，请键入docker-compose down。数据量将持续存在，因此可以使用docker-compose up再次使用相同的数据启动群集。要销毁群集和数据卷，只需键入 docker-compose down -v。

#### 检查群集状态

查询容器IP：

```docker
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' elasticsearch
172.24.0.2
```

检查群集状态：

```bash
curl http://127.0.0.1:9200/_cat/health # 或者使用Docker IP
1529905639 05:47:19 docker-cluster green 2 2 0 0 0 0 0 0 - 100.0%
```

使用docker logs 查看日志。

## 用Docker 配置Elasticsearch

elasticsearch从/usr/share/elasticsearch/config/其下的文件加载它的配置。配置Elasticsearch和设置JVM选项中记录了这些配置文件。

该镜像提供了几种配置Elasticsearch设置的方法，传统方法是提供定制文件，也就是说 elasticsearch.yml。也可以使用环境变量来设置选项：

### 通过Docker环境变量呈现参数

要定义群集名称，docker run您可以通过 -e "cluster.name=mynewclustername"。双引号是必需的。

### 绑定配置

创建您的自定义配置文件并将其安装在图像的相应文件上。例如结合安装一custom_elasticsearch.yml与docker run可与参数来完成：

```docker
-v full_path_to / custom_elasticsearch.yml：/usr/share/elasticsearch/config/elasticsearch.yml
```

容器以用户elasticsearch使用uid：gid的方式运行Elasticsearch1000:1000。绑定安装的主机目录和文件（如custom_elasticsearch.yml上面） 需要由该用户访问。对于数据和日志文件，例如/usr/share/elasticsearch/data写入访问也是必需的。另请参阅下面的注1。

### 自定义镜像

```docker
FROM docker.elastic.co/elasticsearch/elasticsearch:6.3.0
COPY --chown=elasticsearch:elasticsearch elasticsearch.yml /usr/share/elasticsearch/config/
```

然后构建镜像

```docker
docker build --tag=elasticsearch-custom .
docker run -ti -v /usr/share/elasticsearch/data elasticsearch-custom
```

一些插件需要额外的安全权限。您必须通过tty在运行Docker映像时附加一个并在提示中接受yes 来明确接受它们，或者单独检查安全权限，并且如果您对将这些--batch标志添加到plugin install命令时感到满意，那么您必须明确接受它们。查看[插件管理文档](https://www.elastic.co/guide/en/elasticsearch/plugins/6.3/_other_command_line_parameters.html)了解更多详情。

## 使用Elasticsearch Docker镜像配置SSL/TLS

[https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls-docker.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls-docker.html)

## 生产环境注意事项

* 1.默认情况下，Elasticsearch以elasticsearch使用uid：gid的用户身份在容器内运行1000:1000。

```bash
mkdir esdatadir
chmod g+rwx esdatadir
chgrp 1000 esdatadir
```

* 2.确保nofile 和nproc可用于Elasticsearch容器的ulimits是非常重要的

```docker
--ulimit nofile = 65536：65536
```

* 3.交换性能和节点稳定性需要禁用。

```docker
-e“bootstrap.memory_lock = true”--ulimit memlock = -1：-1
```

* 4.该镜像开放 TCP端口9200和9300.对于群集，建议随机化发布的端口--publish-all，除非您为每个主机固定一个容器。

* 5.使用ES_JAVA_OPTS环境变量来设置堆大小。例如，使用16GB的使用-e ES_JAVA_OPTS="-Xms16g -Xmx16g"与docker run。

* 6.`/usr/share/elasticsearch/data`如生产示例中所示， 始终使用绑定的卷，原因如下：

(1) 如果容器被杀死，您的elasticsearch节点的数据不会丢失
(2) Elasticsearch对I/O敏感，而Docker存储驱动程序对于快速I / O并不理想
(3) 它允许使用高级 Docker卷插件

参考：

[https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html)

[Docker环境中Elasticsearch的安装](https://wenchao.ren/archives/250)

[Elasticsearch重要配置](http://www.cnblogs.com/ginb/p/7027910.html)

[ElasticSearch大数据分布式弹性搜索引擎使用](http://www.cnblogs.com/wangiqngpei557/p/5967377.html)

[ElasticSearch入门 第二篇：集群配置](http://www.cnblogs.com/ljhdo/p/4959412.html)

[elasticsearch.yml配置文件](http://www.cnblogs.com/LiZhiW/p/4978396.html)

[https://github.com/13428282016/elasticsearch-CN/wiki/es-setup--elasticsearch](https://github.com/13428282016/elasticsearch-CN/wiki/es-setup--elasticsearch)

[https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html)

[https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/important-settings.html)