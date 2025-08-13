---
title: Docker搭建RabbitMQ高可用群集
date: 2018-07-04 13:45:17
tags:
- 消息队列
- Docker
- RabbitMQ
- 分布式
- 安装部署
categories: 
- RabbitMQ
---
# Docker搭建RabbitMQ高可用群集

官方镜像：

[https://hub.docker.com/_/rabbitmq/](https://hub.docker.com/_/rabbitmq/)
[Dockerfile](https://github.com/docker-library/rabbitmq/blob/4b2b11c59ee65c2a09616b163d4572559a86bb7b/3.7/debian/management/Dockerfile)

## Docker搭建RabbitMQ单机容器

目录结构：

```bash
rabbitmq
|--rabbitmq-a
  |--data
```

获取镜像：

```docker
docker pull rabbitmq:3.7.6-management
```

使用以下命令运行容器：

```docker
docker run -d --name rabbitmq-a \
    -p 4369:4369 \
    -p 5671:5671 \
    -p 5672:5672 \
    -p 25672:25672 \
    -p 15672:15672 \
    -h rabbitmq-node1 \
    -e RABBITMQ_NODENAME=rabbit \
    -e RABBITMQ_DEFAULT_USER=admin \
    -e RABBITMQ_DEFAULT_PASS=admin \
    -v $PWD/rabbitmq-a/data:/var/lib/rabbitmq \
    rabbitmq:3.7.6-management
```

进入容器：

```bash
docker exec -it rabbitmq-a /bin/bash
```

查看群集：

```bash
rabbitmqctl cluster_status
```

结果：

```bash
root@rabbitmq-node1:/# rabbitmqctl cluster_status
Cluster status of node rabbit@rabbitmq-node1 ...
[{nodes,[{disc,['rabbit@rabbitmq-node1']}]},
 {running_nodes,['rabbit@rabbitmq-node1']},
 {cluster_name,<<"rabbit@rabbitmq-node1">>},
 {partitions,[]},
 {alarms,[{'rabbit@rabbitmq-node1',[]}]}]
```

查找cookie:

```bash
find / -name ".erlang.cookie"
cat /var/lib/rabbitmq/.erlang.cookie
```

结果：

```bash
root@rabbitmq-node1:/# find / -name ".erlang.cookie"
/root/.erlang.cookie
/var/lib/rabbitmq/.erlang.cookie
root@rabbitmq-node1:/# cat /var/lib/rabbitmq/.erlang.cookie
BYCWRWRQGDLYZZDXDBSB
```

## RabbitMQ Server 高可用集群相关概念

### 设计集群的目的

* 允许消费者和生产者在 RabbitMQ 节点崩溃的情况下继续运行。
* 通过增加更多的节点来扩展消息通信的吞吐量。

### 集群配置方式

* cluster：不支持跨网段，用于同一个网段内的局域网；可以随意的动态增加或者减少；节点之间需要运行相同版本的 RabbitMQ 和 Erlang。
* federation：应用于广域网，允许单台服务器上的交换机或队列接收发布到另一台服务器上交换机或队列的消息，可以是单独机器或集群。federation 队列类似于单向点对点连接，消息会在联盟队列之间转发任意次，直到被消费者接受。通常使用 federation 来连接 internet 上的中间服务器，用作订阅分发消息或工作队列。
* shovel：连接方式与 federation 的连接方式类似，但它工作在更低层次。可以应用于广域网。

### 节点类型

* RAM node：内存节点将所有的队列、交换机、绑定、用户、权限和 vhost 的元数据定义存储在内存中，好处是可以使得像交换机和队列声明等操作更加的快速。
* Disk node：将元数据存储在磁盘中，单节点系统只允许磁盘类型的节点，防止重启 RabbitMQ 的时候，丢失系统的配置信息。

** 问题说明：**

RabbitMQ 要求在集群中至少有一个磁盘节点，所有其他节点可以是内存节点，当节点加入或者离开集群时，必须要将该变更通知到至少一个磁盘节点。如果集群中唯一的一个磁盘节点崩溃的话，集群仍然可以保持运行，但是无法进行其他操作（增删改查），直到节点恢复。

** 解决方案：**

设置两个磁盘节点，至少有一个是可用的，可以保存元数据的更改。

### 镜像队列

RabbitMQ 的 Cluster 集群模式一般分为两种，普通模式和镜像模式。

* ** 普通模式**：默认的集群模式，以两个节点（rabbit01、rabbit02）为例来进行说明。对于 Queue 来说，消息实体只存在于其中一个节点 rabbit01（或者 rabbit02），rabbit01 和 rabbit02 两个节点仅有相同的元数据，即队列的结构。当消息进入 rabbit01 节点的 Queue 后，consumer 从 rabbit02 节点消费时，RabbitMQ 会临时在 rabbit01、rabbit02 间进行消息传输，把 A 中的消息实体取出并经过 B 发送给 consumer。所以 consumer 应尽量连接每一个节点，从中取消息。即对于同一个逻辑队列，要在多个节点建立物理 Queue。否则无论 consumer 连 rabbit01 或 rabbit02，出口总在 rabbit01，会产生瓶颈。当 rabbit01 节点故障后，rabbit02 节点无法取到 rabbit01 节点中还未消费的消息实体。如果做了消息持久化，那么得等 rabbit01 节点恢复，然后才可被消费；如果没有持久化的话，就会产生消息丢失的现象。
* ** 镜像模式**：将需要消费的队列变为镜像队列，存在于多个节点，这样就可以实现 RabbitMQ 的 HA 高可用性。作用就是消息实体会主动在镜像节点之间实现同步，而不是像普通模式那样，在 consumer 消费数据时临时读取。缺点就是，集群内部的同步通讯会占用大量的网络带宽。

镜像队列实现了 RabbitMQ 的高可用性（HA），具体的实现策略如下所示：

![QQ截图20180705085605.png](/img/QQ截图20180705085605.png)

实例列举：

```bash
queue_args("x-ha-policy":"all") //定义字典来设置额外的队列声明参数
channel.queue_declare(queue="hello-queue",argument=queue_args)
```

如果需要设定特定的节点（以rabbit@localhost为例），再添加一个参数

```bash
queue_args("x-ha-policy":"nodes",
           "x-ha-policy-params":["rabbit@localhost"])
channel.queue_declare(queue="hello-queue",argument=queue_args)
```

可以通过命令行查看那个主节点进行了同步

```bash
rabbitmqctl list_queue name slave_pids synchronised_slave_pids
```

以上内容主要参考：[RabbitMQ分布式集群架构](https://blog.csdn.net/WoogeYu/article/details/51119101)

## Docker-Compose搭建RabbitMQ高可用集群

架构图：

![435188-20180427122219477-216329665.png](/img/435188-20180427122219477-216329665.png)

这个编排主要实现一个磁盘节点、两个内存节点的RabbitMQ集群和一个HAProxy代理。

目录结构：

```bash
rabbitmq
  |--cluster_entrypoint.sh
  |--docker-compose.yml
  |--haproxy.cfg
  |--parameters.env
```

### cluster_entrypoint.sh

```bash
touch cluster_entrypoint.sh ##新建
chmod +x cluster_entrypoint.sh ##执行权限
```

```bash
#!/bin/bash
set -e
if [ -e "/root/is_not_first_time" ]; then
    exec "$@"
else
    /usr/local/bin/docker-entrypoint.sh rabbitmq-server -detached # 先按官方入口文件启动且是后台运行
    rabbitmqctl -n "$RABBITMQ_NODENAME@$RABBITMQ_HOSTNAME" stop_app # 停止应用
    rabbitmqctl -n "$RABBITMQ_NODENAME@$RABBITMQ_HOSTNAME" join_cluster ${RMQHA_RAM_NODE:+--ram} "$RMQHA_MASTER_NODE@$RMQHA_MASTER_HOST" # 加入rabbitmq_node0集群
    rabbitmqctl -n "$RABBITMQ_NODENAME@$RABBITMQ_HOSTNAME" start_app # 启动应用
    rabbitmqctl stop # 停止所有服务
    touch /root/is_not_first_time
    sleep 2s
    exec "$@"
fi
```

注意：
rabbitmqctl join_cluster rabbit@node1
//默认是磁盘节点，如果是内存节点的话，需要加--ram参数

### parameters.env

即公共环境变量

```bash
touch parameters.env
```

```bash
RMQHA_MASTER_NODE=rabbit
RMQHA_MASTER_HOST=rabbitmq_node0
RABBITMQ_DEFAULT_USER=admin
RABBITMQ_DEFAULT_PASS=admin
RABBITMQ_NODENAME=rabbit
RABBITMQ_ERLANG_COOKIE=FQSCWUREIVFXVRTAOIFI
```

### haproxy.cfg

```bash
global
    log     127.0.0.1  local0 info
    log     127.0.0.1  local1 notice
    daemon
    maxconn 4096

defaults
    log     global
    mode    tcp
    option  tcplog
    option  dontlognull
    retries 3
    option  abortonclose
    maxconn 4096
    timeout connect  5000ms
    timeout client  3000ms
    timeout server  3000ms
    balance roundrobin

listen private_monitoring
    bind    0.0.0.0:8100 # haproxy容器8100端口显示代理统计页面
    mode    http
    option  httplog
    stats   enable
    stats   refresh  5s
    stats   hide-version
    stats   uri  /stats
    stats   realm   Haproxy
    stats   auth  admin:admin

listen rabbitmq_admin
    bind    0.0.0.0:15672
    server  rabbitmq_node0 rabbitmq_node0:15672
    server  rabbitmq_node1 rabbitmq_node1:15672
    server  rabbitmq_node2 rabbitmq_node2:15672

listen rabbitmq_cluster
    bind    0.0.0.0:5672
    mode    tcp
    option  tcplog
    balance roundrobin
    timeout client  3h
    timeout server  3h
    server  rabbitmq_node0 rabbitmq_node0:5672  check inter 5s rise 2 fall 3
    server  rabbitmq_node1 rabbitmq_node1:5672  check inter 5s rise 2 fall 3
    server  rabbitmq_node2 rabbitmq_node2:5672  check inter 5s rise 2 fall 3
# ssl for rabbitmq
frontend ssl_rabbitmq
    bind *:5673 ssl crt /usr/local/etc/rmqha.pem
    mode tcp
    default_backend rabbitmq_cluster
```

### docker-compose.yml

注意三个rabbitmq服务，并没有映射主机端口，而是开放容器端口。

```bash
version: "3"
services:
  diskmq:
    image: rabbitmq:3.7.6-management
    container_name: rabbitmq_node0
    restart: always
    hostname: rabbitmq_node0
    ports:
      - "15672"
      - "5672"
    volumes:
      - "./rabbitmq.config:/etc/rabbitmq/conf/rabbitmq.config"
      - "/root/CA:/etc/rabbitmq/conf"
    env_file:
      - ./parameters.env
    environment:
      - CONTAINER_NAME=rabbitmq_node0
      - RABBITMQ_HOSTNAME=rabbitmq_node0
      - RABBITMQ_NODENAME=rabbit
      - RABBITMQ_CONFIG_FILE=/etc/rabbitmq/rabbitmq.config
  rammq1:
    image: rabbitmq:3.7.6-management
    container_name: rabbitmq_node1
    restart: always
    depends_on:
      - diskmq
    hostname: rabbitmq_node1
    ports:
      - "15672"
      - "5672"
    volumes:
      - "./cluster_entrypoint.sh:/usr/local/bin/cluster_entrypoint.sh"
      - "./rabbitmq.config:/etc/rabbitmq/conf/rabbitmq.config"
      - "/root/CA:/etc/rabbitmq/conf"
    entrypoint: "/usr/local/bin/cluster_entrypoint.sh"
    command: "rabbitmq-server"
    env_file:
      - ./parameters.env
    environment:
      - CONTAINER_NAME=rabbitmq_node1
      - RABBITMQ_HOSTNAME=rabbitmq_node1
      - RABBITMQ_NODENAME=rabbit
      - RMQHA_RAM_NODE=true
      - RABBITMQ_CONFIG_FILE=/etc/rabbitmq/rabbitmq.config
  rammq2:
    image: rabbitmq:3.7.6-management
    container_name: rabbitmq_node2
    restart: always
    depends_on:
      - diskmq
    hostname: rabbitmq_node2
    ports:
      - "15672"
      - "5672"
    volumes:
      - "./cluster_entrypoint.sh:/usr/local/bin/cluster_entrypoint.sh"
      - "./rabbitmq.config:/etc/rabbitmq/conf/rabbitmq.config"
      - "/root/CA:/etc/rabbitmq/conf"
    entrypoint: "/usr/local/bin/cluster_entrypoint.sh"
    command: "rabbitmq-server"
    env_file:
      - ./parameters.env
    environment:
      - CONTAINER_NAME=rabbitmq_node2
      - RABBITMQ_HOSTNAME=rabbitmq_node2
      - RABBITMQ_NODENAME=rabbit
      - RMQHA_RAM_NODE=true
      - RABBITMQ_CONFIG_FILE=/etc/rabbitmq/rabbitmq.config
  haproxy:
    image: haproxy:1.8-alpine
    container_name: rabbitmq_proxy
    restart: always
    depends_on:
      - diskmq
      - rammq1
      - rammq2
    hostname: rabbitmq_proxy
    ports:
      - "5672:5672"
      - "5673:5673"
      - "15672:15672"
      - "8100:8100"
    volumes:
      - "./haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg"
    environment:
      - CONTAINER_NAME=rabbitmq_proxy
```

** 执行docker-compose**

运行:

```bash
docker-compose up -d
```

停止：

```bash
docker-compose down
```

### rabbitmq SSL设置

证书生成：

```bash
## 生成CA证书
openssl genrsa -out ca.key 2048
openssl req -passin pass:1111 -new -x509 -days 365 -key ca.key -out ca.crt -subj "/C=CN/ST=JS/L=ZJ/O=zx/OU=test/CN=root"

## 生成服务器端pfx证书
openssl genrsa -passout pass:1111 -des3 -out server.key 2048
openssl req -passin pass:1111 -new -key server.key -out server.csr -subj "/C=CN/ST=JS/L=ZJ/O=zx/OU=test/CN=www.shiyx.top"
openssl x509 -req -passin pass:1111 -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt
openssl pkcs12 -export -out server.pfx -inkey server.key -in server.crt -certfile ca.crt

## haproxy使用ssl证书
cat server.crt server.key|tee rmqha.pem # 将私钥和证书合并到一个文件中
```

参考：

[https://www.rabbitmq.com/ssl.html](https://www.rabbitmq.com/ssl.html)

[https://www.rabbitmq.com/configure.html#customise-environment](https://www.rabbitmq.com/configure.html#customise-environment)

[https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example](https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example)

### rabbitmq镜像队列

mirror queue：镜像队列，queue能在多台机器中同步。

通过webui设置

[QQ截图20180705161425.png](/img/QQ截图20180705161425.png)

或者通过命令行：

```bash
## 表示当前如果是test开头的队列都是“镜像队列”。
rabbitmqctl set_policy ha-all "^test" '{"ha-mode":"all","ha-sync-mode":"automatic"}'
# 或者
rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'
```

### 防火墙开放端口

命令：

```bash
firewall-cmd --zone=public --add-port=8100/tcp --permanent
firewall-cmd --zone=public --add-port=15672/tcp --permanent
firewall-cmd --zone=public --add-port=5672/tcp --permanent
firewall-cmd --reload
```

HAProxy 配置了三个地址：

注：IP地址需要根据实际情况修改

* `http://192.168.7.201:8100/stats` ：HAProxy 负载均衡信息地址，账号密码：admin/admin。
* `http://192.168.7.201:15672`：RabbitMQ Server Web 管理界面（基于负载均衡），账号密码：admin/admin。
* `http://192.168.7.201:5672`：RabbitMQ Server 服务地址（基于负载均衡），账号密码：admin/admin。

## DotNet客户端测试

安装[RabbitMQ.Client](https://www.nuget.org/packages/RabbitMQ.Client)

```bash
Install-Package RabbitMQ.Client -Version 5.1.0
```

```csharp
class Program
{
    static void Main(string[] args)
    {
        ConnectionFactory factory = new ConnectionFactory()
        {
            UserName = "admin",
            Password = "admin",
            AutomaticRecoveryEnabled = true,
            TopologyRecoveryEnabled = true
        };

        //第一步：创建connection 
        var connection = factory.CreateConnection(new string[1] { "192.168.7.201" });

        //第二步：创建一个channel
        var channel = connection.CreateModel();

        var result = channel.QueueDeclare("test1", true, false, false, null);

        for (int i = 0; i < 100; i++)
        {
            channel.BasicPublish(string.Empty, "test1", null, new byte[10]);

            Console.WriteLine("{0} 推送成功", i);
            Thread.Sleep(1000);
        }

        Console.Read();
    }
}
```

支持ssl安全连接：

```csharp
namespace RabbitMqDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            ServicePointManager.ServerCertificateValidationCallback += (sender, cert, chain, sslPolicyErrors) => true;
            ConnectionFactory factory = new ConnectionFactory()
            {
                UserName = "admin",
                Password = "admin",
                AutomaticRecoveryEnabled = true,
                Ssl = new SslOption()
                {
                    CertPath = @"E:\git\RabbitMqDemo\RabbitMqDemo\server.pfx",
                    CertPassphrase = "123123",
                    AcceptablePolicyErrors = SslPolicyErrors.RemoteCertificateNameMismatch |
                                                SslPolicyErrors.RemoteCertificateChainErrors,
                    Enabled = true
                },
                //AuthMechanisms = new AuthMechanismFactory[] { new ExternalMechanismFactory() },
                RequestedHeartbeat = 60,
                Port = 5673,
                TopologyRecoveryEnabled = true
            };
            //第一步：创建connection 
            var connection = factory.CreateConnection(new string[1] { "192.168.0.115" });

            //第二步：创建一个channel
            var channel = connection.CreateModel();

            var result = channel.QueueDeclare("test1", true, false, false, null);

            for (int i = 0; i < int.MaxValue; i++)
            {
                channel.BasicPublish(string.Empty, "test1", null, new byte[10]);

                Console.WriteLine("{0} 推送成功", i);
                //Thread.Sleep(1000);
            }

            Console.Read();
        }
    }
}
```

参考：

[搭建 RabbitMQ Server 高可用集群](https://www.cnblogs.com/xishuai/p/centos-rabbitmq-cluster-and-haproxy.html)

[搭建高可用的rabbitmq集群 + Mirror Queue + 使用C#驱动连接](http://www.cnblogs.com/huangxincheng/p/6113891.html)

[RabbitMQ 高可用集群](https://www.jianshu.com/p/97fbf9c82872)

[RabbitMQ分布式集群架构](https://blog.csdn.net/WoogeYu/article/details/51119101)

[docker搭建rabbitmq集群](https://www.jianshu.com/p/85543491ab00)

[](https://blog.breezelin.site/2017/12/05/practice-rabbitmq-ha-docker-compose/)

[用Docker搭建RabbitMQ高可用集群](https://blog.breezelin.site/2017/12/05/practice-rabbitmq-ha-docker-compose/)

[docker-compose配置rabbitmq集群服务器](https://blog.csdn.net/fqydhk/article/details/80624547)