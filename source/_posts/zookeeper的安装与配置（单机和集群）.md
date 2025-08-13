---
title: zookeeper的安装与配置（单机和集群）
date: 2017-05-31 10:05:13
tags:
- Linux
- CentOS7
- Zookeeper
- 安装部署
- 分布式
- Docker Compose
- Docker
categories:
- Zookeeper 
---

## 单机安装

1、首先去官网下载zookeeper的包 [zookeeper-3.4.10.tar.gz](http://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/ "下载")

2、用FTP文上传到/usr/local下

3、解压文件 `tar -zxvf zookeeper-3.4.10.tar.gz`

4、在conf文件夹下新建zoo.cfg文件，或者使用里面自带的zoo_sample.cfg，重新拷贝

`cp zoo_sample.cfg  zoo.cfg`

zoo.cfg文件内容：

``` linux
tickTime=2000
dataDir=/usr/local/zookeeper/zookeeper-3.4.10/data
dataLogDir=/usr/local/zookeeper/zookeeper-3.4.10/logs
clientPort=4180
```

5、运行server脚本

需要在root账户下启动：

```linux
[root@localhost zookeeper-3.4.10]# ./bin/zkServer.sh start
```

6、Server启动之后, 就可以启动client连接server了, 执行脚本:

```linux
[root@localhost zookeeper-3.4.10]# ./bin/zkCli.sh -server localhost:4180
```

7，检查状态

```linux
[root@localhost zookeeper-3.4.10]# ./bin/zkServer.sh start
```

8，停止

```linux
[root@localhost zookeeper-3.4.10]# ./bin/zkServer.sh stop
```

## 伪集群模式

所谓伪集群, 是指在单台机器中启动多个zookeeper进程, 并组成一个集群. 以启动3个zookeeper进程为例

* 复制三份zookeeper,(权限需要使用root帐号)

```linux
[toor@localhost zookeeper]$ su root
Password:
[root@localhost zookeeper]# cp -r zookeeper-3.4.10 zookeeper-3.4.10-1
[root@localhost zookeeper]# cp -r zookeeper-3.4.10 zookeeper-3.4.10-2
[root@localhost zookeeper]# cp -r zookeeper-3.4.10 zookeeper-3.4.10-3
```

* 配置每个zookeeper文件夹中对应的配置文件

1，zookeeper-3.4.10-1中的zoo.cfg文件

```linux
tickTime=2000
initLimit=5
syncLimit=2
tickTime=2000
dataDir=/usr/local/zookeeper/zookeeper-3.4.10-1/data
dataLogDir=/usr/local/zookeeper/zookeeper-3.4.10-1/logs
clientPort=4180
server.0=127.0.0.1:8880:7770
server.1=127.0.0.1:8881:7771
server.2=127.0.0.1:8882:7772
```

2，zookeeper-3.4.10-2中的zoo.cfg文件

```linux
tickTime=2000
initLimit=5
syncLimit=2
tickTime=2000
dataDir=/usr/local/zookeeper/zookeeper-3.4.10-2/data
dataLogDir=/usr/local/zookeeper/zookeeper-3.4.10-2/logs
clientPort=4180
server.0=127.0.0.1:8880:7770
server.1=127.0.0.1:8881:7771
server.2=127.0.0.1:8882:7772
```

3，zookeeper-3.4.10-3中的zoo.cfg文件

```linux
tickTime=2000
initLimit=5
syncLimit=2
tickTime=2000
dataDir=/usr/local/zookeeper/zookeeper-3.4.10-3/data
dataLogDir=/usr/local/zookeeper/zookeeper-3.4.10-3/logs
clientPort=4180
server.0=127.0.0.1:8880:7770
server.1=127.0.0.1:8881:7771
server.2=127.0.0.1:8882:7772
```

发现只有dataDir和dataLogDir还有clientPort这三个参数不一致，其他参数完全一致。

* 配置myid

    在这三个datadir配置的路径下/Users/zookeeper0、1、2上增加myid文件，里面依次填上0、1、2。

    数字0、1、2和每个conf/zoo.cfg中的server.0、server.1、server.2的数字一一对应，让zookeeper知道你是哪个server

* 依次启动zookeeper节点服务

```linux
./bin/zkServer.sh start
```

检查状态

```linux
./bin/zkServer.sh status
```

客户端连接

```linux
./bin/zkCli.sh -server localhost:4180
```

看到输出显示有下列信息，表示启动，配置成功！

```linux
Welcome to ZooKeeper!
```

## 集群模式

集群模式, 各server部署在不同的机器上, 因此各server的conf/zoo.cfg文件可以完全一样（是所有都一样）

他们的zookeeper的conf下的zoo.cfg文件为：

```linux
tickTime=2000
initLimit=5
syncLimit=2
dataDir=/home/zookeeper/data
dataLogDir=/home/zookeeper/logs
clientPort=4180
server.1=192.168.1.130:2888:3888
server.2=192.168.1.131:2888:3888
server.3=192.168.1.132:2888:3888
```

部署了3台zookeeper server, 分别部署在192.168.1.130, 192.168.1.131, 192.168.1.132上。

各server的dataDir目录下的myid文件中的数字必须不同。

　　192.168.1.130 server的myid为1

　　192.168.1.131 server的myid为2

　　192.168.1.132 server的myid为3

至此，所有的安装与部署就都搞定了。

## ZooKeeper服务命令:

在准备好相应的配置之后，可以直接通过zkServer.sh 这个脚本进行服务的相关操作

1. 启动ZK服务:       sh bin/zkServer.sh start
2. 查看ZK服务状态: sh bin/zkServer.sh status
3. 停止ZK服务:       sh bin/zkServer.sh stop
4. 重启ZK服务:       sh bin/zkServer.sh restart

zoo.cfg配置详解：

tickTime：Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，每个tickTime 时间就会发送一个心跳。

dataDir：Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。

clientPort：客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。

initLimit：Leader和Follower初始化连接时最长能忍受多少个心跳时间间隔数。总的时间长度就是 5*2000=10 秒。

syncLimit：Leader 与 Follower之间发送消息，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 2*2000=4 秒。

server.A=B：C：D：其中 A 是一个数字，表示这个是第几号服务器；B 是这个服务器的 ip 地址；C 表示的是这个服务器与集群中的 Leader 服务器交换信息的端口；D 表示的是万一集群中的 Leader 服务器挂了，需要一个端口来重新进行选举，选出一个新的 Leader，而这个端口就是用来执行选举时服务器相互通信的端口。如果是伪集群的配置方式，由于 B 都是一样，所以不同的 Zookeeper 实例通信端口号不能一样，所以要给它们分配不同的端口号。

myid文件:

除了修改 zoo.cfg 配置文件，集群模式下还要配置一个文件 myid，这个文件在 dataDir 目录下，这个文件里面就有一个数据就是 A 的值，Zookeeper 启动时会读取这个文件，拿到里面的数据与 zoo.cfg 里面的配置信息比较从而判断到底是哪个server。

server.1=10.1.39.43:2888:3888，很多人不理解为啥后面有两个端口？解释一下：

2888：标识这个服务器与集群中的leader服务器交换信息的端口

3888：leader挂掉时专门用来进行选举leader所用的端口

参考：

[http://www.cnblogs.com/hzg110/p/6921886.html](http://www.cnblogs.com/hzg110/p/6921886.html "参考链接")
