---
title: centos7安装Redis-3.2.8
date: 2017-05-16 16:22:36
tags:
- Linux
- CentOS7
- Redis
- 安装部署
categories:
- Redis
---
# centos7安装Redis-3.2.8

## 安装

[官网](https://redis.io/ "官网")下载

下载地址：[redis-3.2.8.tar.gz](http://download.redis.io/releases/redis-3.2.8.tar.gz "redis-3.2.8.tar.gz")

### 依赖包

````linux
yum install gcc
````

### 编译安装

````linux
wget http://download.redis.io/releases/redis-3.2.8.tar.gz # 获取包
tar -zxvf redis-3.2.8.tar.gz
cd redis-3.2.8
cd src
make
make install
````

### 移动文件夹

移动文件，便于管理：(所有源代码安装的软件都安装在/usr/local下)

创建两个文件夹，bin用于存放命令，etc拥有存放配置文件。

````linux
mkdir -p /usr/local/redis/etc
mkdir -p /usr/local/redis/bin
````

-p是递归创建。

接下来，将redis-3.2.8文件夹下的redis.conf复制到/usr/local/redis/etc/

并将src目录下的6个命令文件（绿色的），移动到/usr/local/redis/bin/

````linux
[root@localhost src]# mkdir -p /usr/local/redis/etc
[root@localhost src]# mkdir -p /usr/local/redis/bin
[root@localhost src]# cd ..
[root@localhost redis-3.2.8]# ls
00-RELEASENOTES  COPYING  Makefile   redis.conf       runtest-sentinel  tests
BUGS             deps     MANIFESTO  runtest          sentinel.conf     utils
CONTRIBUTING     INSTALL  README.md  runtest-cluster  src
[root@localhost redis-3.2.8]# mv ./redis.conf /usr/local/redis/etc
[root@localhost redis-3.2.8]# ls
00-RELEASENOTES  COPYING  Makefile   runtest           sentinel.conf  utils
BUGS             deps     MANIFESTO  runtest-cluster   src
CONTRIBUTING     INSTALL  README.md  runtest-sentinel  tests
[root@localhost redis-3.2.8]# cd src
[root@localhost src]# mv mkreleasehdr.sh redis-benchmark redis-check-aof redis-check-dump redis-cli redis-sentinel redis-server /usr/local/redis/bin/
````

### 启动redis服务

```linux
[toor@localhost redis]$ cd bin
[toor@localhost bin]$ ./redis-server
```

显示结果：

![结果](/img/redis-3.2.8安装.png)

但是，这样做的话，我们并没有使用etc的下的配置文件进行启动（图中红线部分）。

如果希望通过指定的配置文件启动，需要在启动时指定配置文件：

使用ctrl+C终止服务(显示错误，可以重试几次)，然后查看redis服务是否终止干净了，之后通过设置配置文件来启动服务

运行：`pstree -p | grep redis` 发现redis服务已经被终止干净

现在我们带上配置文件 /usr/local/etc/redis.conf 运行redis

```linux
[toor@localhost bin]$  ./redis-server /usr/local/redis/etc/redis.conf
```

启动结果：
![启动结果](/img/redis-3.2.8运行结果.png)

这样启动redis仍然在前台运行；

redis后台运行需要将daemonize由no改为yes

首先在超级权限下打开redis.conf。

```linux
[root@localhost bin]# vim /usr/local/redis/etc/redis.conf
```

保存退出，然后再次使用配置文件启动redis-server

```linux
[root@localhost bin]# ./redis-server /usr/local/redis/etc/redis.conf
[root@localhost bin]# ps -ef|grep redis
root     101598      1  0 17:04 ?        00:00:00 ./redis-server 127.0.0.1:6379
root     101607 101338  0 17:04 pts/0    00:00:00 grep --color=auto redis
[root@localhost bin]# pstree -p|grep redis
           |-redis-server(101598)-+-{redis-server}(101601)
           |                      `-{redis-server}(101602)
```

详细参数说明：

daemonize 如果需要在后台运行，把该项改为yes

pidfile 配置多个pid的地址 默认在/var/run/redis.pid

bind 绑定ip，设置后只接受来自该ip的请求

port 监听端口，默认是6379

loglevel 分为4个等级：debug verbose notice warning

logfile 用于配置log文件地址

databases 设置数据库个数，默认使用的数据库为0

save 设置redis进行数据库镜像的频率。

rdbcompression 在进行镜像备份时，是否进行压缩

dbfilename 镜像备份文件的文件名

Dir 数据库镜像备份的文件放置路径

Slaveof 设置数据库为其他数据库的从数据库

Masterauth 主数据库连接需要的密码验证

Requriepass 设置 登陆时需要使用密码

Maxclients 限制同时使用的客户数量

Maxmemory 设置redis能够使用的最大内存

Appendonly 开启append only模式

Appendfsync 设置对appendonly.aof文件同步的频率（对数据进行备份的第二种方式）

vm-enabled 是否开启虚拟内存支持   （vm开头的参数都是配置虚拟内存的）

vm-swap-file 设置虚拟内存的交换文件路径

vm-max-memory 设置redis使用的最大物理内存大小

vm-page-size 设置虚拟内存的页大小

vm-pages 设置交换文件的总的page数量

vm-max-threads 设置VM IO同时使用的线程数量

Glueoutputbuf 把小的输出缓存存放在一起

hash-max-zipmap-entries 设置hash的临界值

Activerehashing 重新hash

Redis服务端默认连接端口是6379.

mysql 服务端默认连接端口是3306

Mogodb 服务端默认连接端口是27017，还有28017。

在平时，我们往往需要查看6379端口是否被占用。可以用以下命令：

````linux
[toor@localhost bin]$ netstat -tunpl | grep 6379
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      -
[toor@localhost bin]$ sudo netstat -tunpl | grep 6379
[sudo] password for toor:
toor is not in the sudoers file.  This incident will be reported.
[toor@localhost bin]$ su root
Password:
[root@localhost bin]# netstat -tunpl | grep 6379
tcp        0      0 127.0.0.1:6379          0.0.0.0:*               LISTEN      101598/./redis-serv 
````

注意，redis服务需要su权限才能查看，不然只能检查到6379被某个进程占用，但是看不到进程名称。

至此，redis服务已经按照配置文件启动成功！！

### 客户端登陆

进入 `/usr/local/redis/bin/`

```linux
[root@localhost bin]# redis-cli
127.0.0.1:6379>

[root@localhost bin]# redis-cli -h 192.168.1.131 -p 6379 # 指定主机，端口 -a password #指定密码
192.168.1.131:6379>
[root@localhost bin]# redis-cli -h 192.168.1.131 -p 6379 shutdown # 关闭redis服务
```

### 关闭Redis服务

```linux
[root@localhost bin]# pkill redis-server
[root@localhost bin]# netstat -tunpl | grep 6379
[root@localhost bin]# pstree -p | grep redis
[root@localhost bin]# redis-cli
Could not connect to Redis at 127.0.0.1:6379: Connection refused
Could not connect to Redis at 127.0.0.1:6379: Connection refused
not connected>
```

也可以使用/usr/local/redis/bin/redis-cli shutdown，这种方法使用客户端命令redis-cli

```linux
[root@localhost bin]# ./redis-server /usr/local/redis/etc/redis.conf
[root@localhost bin]# pstree -p | grep redis
           |-redis-server(101927)-+-{redis-server}(101929)
           |                      `-{redis-server}(101930)
[root@localhost bin]# /usr/local/redis/bin/redis-cli shutdown
[root@localhost bin]# pstree -p | grep redis
[root@localhost bin]# sudo netstat -tunpl | grep 6379
```

### 开启远程访问端口

    > 开启端口：

````linux
firewall-cmd --zone=public --add-port=6379/tcp --permanent
````

    命令含义：

````linux
--zone #作用域

--add-port=27017/tcp #添加端口，格式为：端口/通讯协议

--permanent #永久生效，没有此参数重启后失效

firewall-cmd --reload  #重启防火墙
````

### 配置密码认证

找到/usr/local/redis/etc/redis.conf文件中

```linux
#requirepass foobared
```

去掉注释，并且设置密码

重启redis

````linux
[root@localhost bin]# ./redis-server /usr/local/redis/etc/redis.conf
[root@localhost bin]# redis-cli -h 192.168.1.131 -p 6379 -a toor
192.168.1.131:6379> keys *
(empty list or set)
````

````linux
[root@localhost bin]# ./redis-server /usr/local/redis/etc/redis.conf
[root@localhost bin]# redis-cli -h 192.168.1.131 -p 6379
192.168.1.131:6379> keys *
(error) NOAUTH Authentication required.
192.168.1.131:6379> auth toor
OK
192.168.1.131:6379> keys *
(empty list or set)
````

master配置了密码，slave如何配置

若master配置了密码则slave也要配置相应的密码参数否则无法进行正常复制的。

slave中配置文件内找到如下行，移除注释，修改密码即可

```linux
#masterauth  mstpassword
```

参考：

[http://www.open-open.com/lib/view/open1426468117367.html](http://www.open-open.com/lib/view/open1426468117367.html "参考链接")