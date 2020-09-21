---
title: CentOS7 安装MongoDB
date: 2017-05-10 10:43:53
tags:
- Linux
- CentOS7
- MongoDB
- 安装部署
categories:
- MongoDB
---

## 流程

### 下载

[官网下载网址](https://www.mongodb.com/download-center?jmp=nav#community "链接") 

找到CentOS7最新下载包，

[mongodb-linux-x86_64-rhel70-3.4.4.tgz](https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-3.4.4.tgz "下载链接")

[官方手册](https://docs.mongodb.com/manual/ "链接") 

### 解压

在终端输入命令,如下：

```sh
# 解压
tar xzvf mongodb-linux-x86_64-rhel70-3.4.4.tgz

# 移动
mv mongodb-linux-x86_64-rhel70-3.4.4 mongodb-3.4.4

# 创建目录
cd mongodb-3.4.4
mkdir logs
mkdir db

# 创建配置文件
cd bin
vim mongodb.conf
```

**配置文件如下配置：**

```sh
# idae - MongoDB config start - 2017-05-10

# 设置数据文件的存放目录
dbpath = /usr/local/mongodb/mongodb-3.4.4/db

# 设置日志文件的存放目录及其日志文件名
logpath = /usr/local/mongodb/mongodb-3.4.4/logs/mongodb.log

# 设置端口号（默认的端口号是 27017）
port = 27017

# 设置为以守护进程的方式运行，即在后台运行
fork = true

# nohttpinterface = true
nohttpinterface = true
# idae - MongoDB config end - 2016-05-10
```

**参数解释**

```linux
--dbpath 数据库路径(数据文件)
--logpath 日志文件路径
--master 指定为主机器
--slave 指定为从机器
--source 指定主机器的IP地址
--pologSize 指定日志文件大小不超过64M.因为resync是非常操作量大且耗时，最好通过设置一个足够大的oplogSize来避免resync(默认的 oplog大小是空闲磁盘大小的5%)。
--logappend 日志文件末尾添加，即使用追加的方式写日志
--journal 启用日志
--port 启用端口号
--fork 在后台运行
--only 指定只复制哪一个数据库
--slavedelay 指从复制检测的时间间隔
--auth 是否需要验证权限登录(用户名和密码)
--syncdelay 数据写入硬盘的时间（秒），0是不等待，直接写入
--notablescan 不允许表扫描
--maxConns 最大的并发连接数，默认2000  
--pidfilepath 指定进程文件，不指定则不产生进程文件
--bind_ip 绑定IP，绑定后只能绑定的IP访问服务（服务器本地IP，非访问服务器客户端IP）
```

**启动mongodb服务**

```sh
# 进入mongodb-3.4.4/bin目录
# 自定义的 mongodb 配置文件方式启动
./mongod --config mongodb.conf

# 停止:
./mongod --config mongodb.conf --shutdown

# 修复模式启动 mongodb
./mongod --repair -f mongodb.conf

# 以参数方式启动（在mongodb-3.4.4/bin目录下）:
./mongod --dbpath=/usr/local/mongodb/mongodb-3.4.4/db --logpath=/usr/local/mongodb/mongodb-3.4.4/logs/mongodb.log --fork

# 停止：添加 --shutdown
./mongod --dbpath=/usr/local/mongodb/mongodb-3.4.4/db --logpath=/usr/local/mongodb/mongodb-3.4.4/logs/mongodb.log --fork --shutdown

# 添加认证： --auth
# 连接
./mongo 192.168.1.131:27017/admin -u root1 -p root1

# 如果报如下错误：

ERROR: child process failed, exited with error number 1
# 很可能是 mongodb.conf 中配置的路径不一致问题；

# 如果报如下错误：

ERROR: child process failed, exited with error number 100
# 很可能是没有正常关闭导致的，那么可以删除 mongod.lock 文件
# number 100 ：端口问题
```

**查看 mongodb 进程：**

`ps aux | grep mongodb`

**查看 mongodb 服务的运行日志：**

 `tail -200f /usr/local/mongodb/mongodb-3.4.4/logs/mongodb.log`

**检查端口是否已被启动：**

`netstat -lanp | grep 27017`

**杀死 mongodb 进程，即可关闭 mongodb 服务：(禁止使用，容易出现问题)**

```sh
kill -15 PID
```

**连接错误**

1.若数据库出现如上不能连接的原因，可能是data目录下的mongod.lock文件问题，可以用如下命令修复：
`./bin/mongod --repair`

2.直接删除mongod.lock
`rm -f /usr/local/mongodb/db/mongod.lock`

3.然后再启动 mongodb 服务：
`./mongod --config mongodb.conf`

4.如果以上两部依然解决不掉，则是路径文件，我们可以删除 `/usr/local/mongodb/mongodb-3.4.4/db` 目录及其子目录，并采用绝对路径的方式：

```sh
./mongod /usr/local/mongodb/mongodb-3.4.4/bin/mongod --dbpath=/usr/local/mongodb/mongodb3.4.4/db --logpath=/usr/local/mongodb/mongodb-3.4.4/logs/mongodb.log --fork
```

**将 mongodb 服务加入到自启动文件中：**

`vi /etc/rc.local`

在文件末尾追加如下命令：

`/usr/local/mongodb/mongodb-3.4.4/bin/mongod --config mongodb.conf`

保存并退出：
:wq!

在 `/usr/local/mongodb/mongodb3.4.4/bin/` 目录中，键入如下命令，打开一个 mongodb 的客户端程序，即打开一个 mongodb 的 shell 客户端，这个 shell 客户端同时也是一个 JavaScript 编辑器，即可用输入任何的 JavaScript 脚本：

`./mongo`

在浏览器中输入 IP:27017，如：

`http://127.0.0.1:27017/`

**开启远程访问端口**

开启端口：

`firewall-cmd --zone=public --add-port=27017/tcp --permanent`

命令含义：
--zone #作用域
--add-port=27017/tcp #添加端口，格式为：端口/通讯协议
--permanent #永久生效，没有此参数重启后失效

重启防火墙:

`firewall-cmd --reload`

### 安全和认证

**限制特定IP地址访问(MongoDB服务器IP地址)**

```sh
./mongod --bind_ip 192.168.1.131 --port 27017 --dbpath /usr/local/mongodb/mongodb-3.4.4/db --logpath /usr/local/mongodb/mongodb-3.4.4/logs/mongodb.log --fork
```

**设置监听端口，并且需要在防火墙配置**

```sh
--port 27017
```

**设置登录用户名和口令**

启用mongodb授权认证的方法：

1、以–auth 启动mongod

2、在配置文件mongod.conf 中加入 auth = true

添加用户：

```linux
db.createUser(
...   {
...     user: "root1",
...     pwd: "root1",
...     roles: [ { role: "__system", db: "admin" } ]
...   }
... )
```
重新登录：
`./mongo 192.168.1.131:27017 -u root1 -p root1`

展示角色：
`show roles`

获取用户：
`db.getUsers();`

参考网址：

[http://www.linuxidc.com/Linux/2016-06/132675.htm](http://www.linuxidc.com/Linux/2016-06/132675.htm "链接")