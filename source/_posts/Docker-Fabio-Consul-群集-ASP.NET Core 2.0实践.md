---
title: Docker & Fabio & Consul群集 & ASP.NET Core 2.0实践
date: 2018-06-13 11:36:36
tags:
  - Docker
  - Consul
  - DotNetCore
  - Fabio
  - 分布式
categories: 
  - Consul
---

## 1，Docker运行consul环境

### 1、Consul基础介绍

[https://www.consul.io/](https://www.consul.io/)

Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其他分布式服务注册与发现的方案，比如 Airbnb 的 SmartStack 等相比，Consul 的方案更“一站式”，一致性协议采用 Raft 算法,用来保证服务的高可用. 使用 GOSSIP 协议管理成员和广播消息, 并且支持 ACL 访问控制，内置了服务注册与发现框架、分布一致性协议实现、健康检查,并允许 HTTP 和 DNS 协议调用 API 存储键值对、Key/Value 存储、多数据中心方案，不再需要依赖其他工具（比如 ZooKeeper 等）。使用起来也较为简单。
Consul 用 Golang 实现，因此具有天然可移植性(支持 Linux、windows 和 Mac OS X)；安装包仅包含一个可执行文件，方便部署，与 Docker 等轻量级容器可无缝配合。

要想利用Consul提供的服务实现服务的注册与发现，我们需要建立Consul Cluster。在Consul方案中，每个提供服务的节点上都要部署和运行Consul的agent，所有运行Consul Agent节点的集合构成Consul Cluster。Consul Agent有两种运行模式：Server和Client。这里的Server和Client只是Consul集群层面的区分，与搭建在Cluster之上的应用服务无关。以Server模式运行的Consul Agent节点用于维护Consul集群的状态，官方建议每个Consul Cluster至少有3个或以上的运行在Server Mode的Agent，Client节点不限。

　　Consul支持多数据中心，每个数据中心的Consul Cluster都会在运行于Server模式下的Agent节点中选出一个Leader节点，这个选举过程通过Consul实现的raft协议保证，多个Server节点上的Consul数据信息是强一致的。处于Client Mode的Consul Agent节点比较简单，无状态，仅仅负责将请求转发给Server Agent节点。

** Consul 功能：**

* 服务发现（Service Discovery）：客户端通过 Consul 提供服务，其他客户端可以通过 Consul 利用 dns 或者 http 发现依赖服务

* 健康检查（Health Checking）: Consul 提供任务的健康检查，可以用来操作或者监控集群的健康，也可以在服务发现时去除失效的服务

* 键值对存储（Key/Value Store）: 存储层级键值对

* 多数据中心（Multi Datacenter）: Consul 支持开箱即用的多数据中心

** Consul的优势**

* 使用 Raft 算法来保证一致性, 比复杂的 Paxos 算法更直接. 相比较而言, zookeeper 采用的是 Paxos, 而 etcd 使用的则是 Raft.

* 支持多数据中心，内外网的服务采用不同的端口进行监听。 多数据中心集群可以避免单数据中心的单点故障,而其部署则需要考虑网络延迟, 分片等情况等. zookeeper 和 etcd 均不提供多数据中心功能的支持.

* 支持健康检查. etcd 不提供此功能.

* 支持 http 和 dns 协议接口. zookeeper 的集成较为复杂, etcd 只支持 http 协议,官方提供web管理界面, etcd 无此功能.

** Consul 的使用场景**

* docker 实例的注册与配置共享

* coreos 实例的注册与配置共享

* vitess 集群

* SaaS 应用的配置共享

* 与 confd 服务集成，动态生成 nginx 和 haproxy 配置文件

** Consul 的模式**

* client: 客户端, 无状态, 将 HTTP 和 DNS 接口请求转发给局域网内的服务端集群。一个 Client 是一个转发所有 RPC 到 Server 的代理。这个 Client 是相对无状态的；Client 唯一执行的后台活动是加入 LAN gossip 池，这有一个最低的资源开销并且仅消耗少量的网络带宽。

* server: 服务端, 保存配置信息, 高可用集群, 在局域网内与本地客户端通讯, 通过广域网与其他数据中心通讯. 每个数据中心的 server 数量推荐为 3 个或是 5 个。参与 Raft 选举，维护集群状态，响应 RPC 查询，与其他数据中心交互 WAN gossip 和转发查询给 leader 或者远程数据中心。

群集架构图如下：

![n4mdw.jpg](/img/n4mdw.jpg)

![435188-20161228110501664-588566717.png](/img/435188-20161228110501664-588566717.png)

Consul和其他类似软件的对比：

![QQ截图20180613135320.png](/img/QQ截图20180613135320.png)

### 2、Consul群集搭建

Consul 集群搭建时一般提供两种模式:

** 手动模式**: 启动第一个节点后，此时此节点处于 bootstrap 模式，其节点手动执行加入，使用： -join命令
** 自动模式**: 启动第一个节点后，在其他节点配置好尝试加入的目标节点，然后等待其自动加入(不需要人为命令加入)，配置：retry_join

以自动模式为例：

目录结构：

``` bash
document
|--consul
   |--server1.json
   |--server2.json
   |--server3.json
   |--client1.json
   |--data
```

server1.json配置文件：

```json
{
  "data_dir": "/data",
  "datacenter": "consul-test",
  "log_level": "INFO",
  "node_name": "server1",
  "server": true,
  "bind_addr": "172.17.0.2",
  "bootstrap": true,
  "retry_join": ["172.17.0.2", "172.17.0.3", "172.17.0.4"]
}
```

server2.json配置文件：

```json
{
  "data_dir": "/data",
  "datacenter": "consul-test",
  "log_level": "INFO",
  "node_name": "server2",
  "server": true,
  "bind_addr": "172.17.0.3",
  "retry_join": ["172.17.0.2", "172.17.0.3", "172.17.0.4"]
}
```

server3.json配置文件：

```json
{
  "data_dir": "/data",
  "datacenter": "consul-test",
  "log_level": "INFO",
  "node_name": "server3",
  "server": true,
  "bind_addr": "172.17.0.4",
  "retry_join": ["172.17.0.2", "172.17.0.3", "172.17.0.4"]
}
```

client1.json配置文件：

```json
{
    "data_dir": "/data",
    "datacenter": "consul-test",
    "log_level": "INFO",
    "node_name": "client1",
    "server": false,
    "ui": true,                         // 是否开启 UI 访问
    "http_config": {
        "response_headers": {
          "Access-Control-Allow-Origin": "*"
        }
    },
    "addresses": {
        "http": "0.0.0.0"
    },
    "start_join": ["172.17.0.2", "172.17.0.3", "172.17.0.4"]
}
```

运行docker容器：

```bash
docker run --name=server1 -it -d -v $PWD/consul:/consul consul agent -config-file=/consul/server1.json

docker run --name=server2 -it -d -v $PWD/consul:/consul consul agent -config-file=/consul/server2.json

docker run --name=server3 -it -d -v $PWD/consul:/consul consul agent -config-file=/consul/server3.json

docker run --name=client1 -it -d -v $PWD/consul:/consul consul agent -config-file=/consul/client1.json
```

查看容器运行日志(选择其中一个即可)，可以看出选举server1作为Leader：

```bash
docker logs server1
```

结果：

```bash
2018/06/14 08:12:06 [INFO] consul: New leader elected: server1
```

可以查看容器的ip:

```bash
docker inspect -f '{{.NetworkSettings.IPAddress}}' server1
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' server1
```

查看下集群的状态

注：server1为Leader;

```bash
docker exec -t server1 consul members
```

结果：

```bash
[root@localhost document]# docker exec -t server1 consul members
Node     Address          Status  Type    Build  Protocol  DC           Segment
server1  172.17.0.2:8301  alive   server  1.1.0  2         consul-test  <all>
server2  172.17.0.3:8301  alive   server  1.1.0  2         consul-test  <all>
server3  172.17.0.4:8301  alive   server  1.1.0  2         consul-test  <all>
client1  172.17.0.5:8301  alive   client  1.1.0  2         consul-test  <default>
```

访问：[http://172.17.0.5:8500](http://172.17.0.5:8500),打开 Consul UI 界面，就可以看到我们配置的 Consul Client:

![QQ截图20180615092803.png](/img/QQ截图20180615092803.png)

可以使用以下命令查询其它信息：

```bash
curl -L http://172.17.0.5:8500/v1/catalog/datacenters
curl -L http://172.17.0.5:8500/v1/agent/members | python -m json.tool
curl http://172.17.0.5:8500/v1/agent/self | python -m json.tool
```

### 3、节点异常处理

#### LEADER 挂了

leader挂了，consul会重新选取出新的leader，只要超过一半的SERVER还活着，集群是可以正常工作的。

```bash
docker stop server1
```

![QQ截图20180615095815.png](/img/QQ截图20180615095815.png)

运行以下命令查看现在的Leader：

```bash
curl http://172.17.0.5:8500/v1/status/leader
```

结果：

```bash
[root@localhost document]# curl http://172.17.0.5:8500/v1/status/leader
"172.17.0.3:8300"
```

注：172.17.0.3为server2 IP。

## 2，Docker 运行 Fabio 环境

### Fabio简介

[Fabio](https://github.com/fabiolb/fabio) 是 ebay 团队用 golang 开发的一个快速、现代、zero-conf 负载均衡 HTTP(S) 路由器，用于部署 Consul 管理的微服务。

因为 consul 支持服务注册与健康检查所以 fabio 能够零配置提供负载。

根据项目的介绍fabio 能提供每秒15000次请求。

**服务发现的特点**

服务与服务之间的调用需要在配置文件中填写好主机和端口,不易于维护且分布式环境中不易于部署与扩容。

那么此时就需要考虑服务启动的时候自己把主机和端口以及一些其他信息注册到注册中心，这样其他服务可以从中找到它。

甚至更为简单的注册完毕后通过 DNS 的方式来『寻址』。比如 Zookeepr 可以很好的完成这个工作，但是其中还有一个弊端就是服务的健康检查服务注册到注册中心之后如何保证这个服务一定可用？此时就需要自己来写逻辑当服务不可用的时候自动从注册中心下线掉。 然后Consul 可以很轻易的解决这个问题。

**工作原理**

Consul 提供了一套健康检测机制简单的说针对 http 类型的服务(consul 也支持 其他类型例如tcp)在注册的时候可以顺便注册下健康检测的信息，提供一个健康检测的地址(url)以及一个频率超时时间这样的话 consul 会定期的来请求当状态码是200的时候设置次服务是健康的状态否则是故障状态。

既然注册到consul的服务能够自己维护健康状态此时 fabio 的工作就很简单了！ 就是直接从consul 注册表里面取出健康的服务根据服务注册时候的 tags 配置自动创建自己的路由表，然后当一个 http 请求过来的时候自动去做负载均衡

简单流程图：

```fabio
======    服务注册     =========       =========
A服务   <------>      consul集群  ---->  健康的 A/不健康的 A 集群
======    健康检查     =========       =========
                                     ^
                                     | 加入/移出路由表
                                     |
                                  ========
                                   fabio 集群
                                  ========
                                     ^
                                     |
                                     | A服务   如果找到则成功路由否则返回错误
                                     V
                                    http 请求
```

### Docker 运行 Fabio

Fabio Docker 镜像地址：[https://hub.docker.com/r/magiconair/fabio/](https://hub.docker.com/r/magiconair/fabio/)

**目录结构**

```bash
fabio
|--fabio.properties
```

首先，在/fabio/目录下创建一个fabio.properties文件（[示例配置](https://raw.githubusercontent.com/eBay/fabio/master/fabio.properties)），然后vim fabio.properties增加下面配置：

```fabio
registry.consul.register.addr = 172.17.0.6:9998
registry.consul.addr = 172.17.0.5:8500
```

在fabio文件夹下运行命令：

```bash
docker run -d -p 9999:9999 -p 9998:9998 --name=fabio -v $PWD/fabio.properties:/etc/fabio/fabio.properties magiconair/fabio
```

注：默认是bridge网络模式，也就是我们常用的桥接模式，Docker 会分配给容器一个独立的 IP 地址（端口也是独立的），并且容器和主机之间可以相互访问。或者添加参数：--net=host：host网络模式，容器的网络接口和主机一样，也就是共享一个 IP 地址。

![QQ截图20180615172806.png](/img/QQ截图20180615172806.png)

## 3，Docker 运行 ASP.NET Core 2.0 服务

### 3.1 准备asp.net Core应用程序

![QQ截图20180620090937.png](/img/QQ截图20180620090937.png)

### 3.2 添加HealthController用于Consul健康检查

``` csharp
    [Produces("application/json")]
    [Route("api/Health")]
    public class HealthController : Controller
    {
        [HttpGet]
        public IActionResult Get() => Ok("ok");
    }
```

### 3.3 调用Consul API注册服务

组件地址：[https://github.com/PlayFab/consuldotnet](https://github.com/PlayFab/consuldotnet)

（1） 首先安装 Consul包：

```dotnet
install-package Conusl
```

（2） 添加扩展方法，用于调用Consul API

```csharp
public static class AppBuilderExtensions
    {
        public static IApplicationBuilder RegisterWithConsul(this IApplicationBuilder app, IApplicationLifetime lifetime, ServiceEntity serviceEntity)
        {
            var consulClient = new ConsulClient(x => x.Address = new Uri($"http://{serviceEntity.ConsulIP}:{serviceEntity.ConsulPort}"));//请求注册的 Consul 地址
            var httpCheck = new AgentServiceCheck()
            {
                DeregisterCriticalServiceAfter = TimeSpan.FromSeconds(5),//服务启动多久后注册
                Interval = TimeSpan.FromSeconds(10),//健康检查时间间隔，或者称为心跳间隔

                // 即本程序运行后的IP地址。需要使用命令：
                // docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' consultest_consultest_1
                HTTP = $"http://{serviceEntity.IP}:{serviceEntity.Port}/api/health",//健康检查地址
                Timeout = TimeSpan.FromSeconds(5)
            };

            // Register service with consul
            var registration = new AgentServiceRegistration()
            {
                Checks = new[] { httpCheck },
                ID = Guid.NewGuid().ToString(),
                Name = serviceEntity.ServiceName,
                Address = serviceEntity.IP,
                Port = serviceEntity.Port,
                Tags = new[] { $"urlprefix-/{serviceEntity.ServiceName}" }//添加 urlprefix-/servicename 格式的 tag 标签，以便 Fabio 识别
            };

            consulClient.Agent.ServiceRegister(registration).Wait();//服务启动时注册，内部实现其实就是使用 Consul API 进行注册（HttpClient发起）
            lifetime.ApplicationStopping.Register(() =>
            {
                consulClient.Agent.ServiceDeregister(registration.ID).Wait();//服务停止时取消注册
            });
            return app;
        }
    }
```

（3）Starup类的Configure方法中，调用此扩展方法

```csharp
 // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
        public void Configure(IApplicationBuilder app, IHostingEnvironment env, IApplicationLifetime lifetime)
        {
            if (env.IsDevelopment())
            {
                app.UseBrowserLink();
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
            }

            app.UseStaticFiles();

            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });

            // register this service
            ServiceEntity serviceEntity = new ServiceEntity
            {
                IP = Configuration["Service:IP"],// NetworkHelper.LocalIPAddress,
                Port = Convert.ToInt32(Configuration["Service:Port"]),
                ServiceName = Configuration["Service:Name"],
                ConsulIP = Configuration["Consul:IP"],
                ConsulPort = Convert.ToInt32(Configuration["Consul:Port"])
            };
            app.RegisterWithConsul(lifetime, serviceEntity);
        }
```

ServiceEntity类定义:

```csharp
public class ServiceEntity
    {
        public string IP { get; set; }
        public int Port { get; set; }
        public string ServiceName { get; set; }
        public string ConsulIP { get; set; }
        public int ConsulPort { get; set; }
    }
```

appSettings.json配置文件:

```json
{
  "Service": {
    "IP": "172.21.0.2",
    "Name": "ConsulDemo",
    "Port": "80"
  },
  "Consul": {
    "IP": "172.17.0.5",
    "Port": "8500"
  },
  "Logging": {
    "IncludeScopes": false,
    "LogLevel": {
      "Default": "Warning"
    }
  }
}
```

### 3.4 运行ASP.NET Core程序

在Linux服务器下，建立程序文件夹，使用git clone命令下载代码，如：

```bash
mkdir web
cd web
git clone https://github.com/syxdevcode/ConsulTest.git
git pull origin master ### 更新代码
```

使用docker-compose 运行程序：

```bash
cd ConsulTest ## 进入目录
docker-compose up -d --build
```

结果：

```bash
Successfully built dfaf37c4246f
Successfully tagged consultest:latest
Recreating consultest_consultest_1 ... done
```

查看docker 容器：

```docker
docker ps -a
```

结果：

![QQ截图20180620092619.png](/img/QQ截图20180620092619.png)

其它命令：

查看容器IP：

```bash
docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' consultest_consultest_1
```

### 3.5 验证

![QQ截图20180620093002.png](/img/QQ截图20180620093002.png)

![QQ截图20180620093047.png](/img/QQ截图20180620093047.png)

参考：

[https://github.com/PlayFab/consuldotnet](https://github.com/PlayFab/consuldotnet)

[https://www.consul.io/docs/agent/options.html](https://www.consul.io/docs/agent/options.html)

[.NET Core微服务之基于Consul实现服务治理](http://www.cnblogs.com/edisonchou/p/9124985.html)

[Mac OS、Ubuntu 安装及使用 Consul](http://www.cnblogs.com/xishuai/p/macos-ubuntu-install-consul.html)

[Docker & Consul & Fabio & ASP.NET Core 2.0 微服务跨平台实践](http://www.cnblogs.com/xishuai/p/ubuntu-docker-consul-fabio-aspnet-core.html)

[Docker 三剑客之 Docker Compose](http://www.cnblogs.com/xishuai/p/docker-compose.html)

[Consul的安装方法](http://xiaoquqi.github.io/blog/2015/12/07/consul-installation/)

[Consul 基础](http://soft.dog/2016/03/18/consul-basic/)

[Consul 原理和使用简介](https://blog.coding.net/blog/intro-consul)

[Consul 集群搭建](https://mritd.me/2017/09/21/set-up-ha-consul-cluster/)

[Consul集群部署](http://www.10tiao.com/html/357/201705/2247485185/1.html)

[服务发现 - consul 的介绍、部署和使用](https://www.jianshu.com/p/f8746b81d65d)

[Consul + fabio 实现自动服务发现、负载均衡](http://dockone.io/article/1567)