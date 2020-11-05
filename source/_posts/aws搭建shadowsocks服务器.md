---
title: aws搭建shadowsocks服务器
date: 2018-04-24 9:12:23
tags:
- Linux
- AWS
- Ubuntu
- Shadowsocks
- 安装部署
categories:
- Shadowsocks
---
# aws搭建shadowsocks服务器

## shadowsocks特点：

地址：
[https://github.com/shadowsocks/shadowsocks](https://github.com/shadowsocks/shadowsocks "链接")

快速（异步I/O和事件驱动程序）

安全（所有的流量都经过加密算法加密，支持自定义算法）

支持移动客户端（专为移动设备和无线网络优化）

跨平台（可运行于包括PC，Mac，手机（Android和iOS）和路由器（OpenWrt）在内的多种平台上）

## 安装shadowsocks

[参考官网命令](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E "链接")

Debian / Ubuntu:

```sh
sudo apt update 
sudo apt install python2
curl https://bootstrap.pypa.io/get-pip.py --output get-pip.py
sudo python2 get-pip.py
pip2 --version

pip install shadowsocks
```

CentOS:

```sh
yum install python-setuptools && easy_install pip
pip install shadowsocks
```

## 配置shadowsocks

无论是centos系统还是ubuntu系统，shadowsocks配置都是一样的。

shadowsocks安装完毕后，可以查看使用ssserver命令进行查看

```sh
ubuntu@ip-172-31-16-20:~$ ssserver -h
usage: ssserver [OPTION]...
A fast tunnel proxy that helps you bypass firewalls.

You can supply configurations via either config file or command line arguments.

Proxy options:
  -c CONFIG              path to config file
  -s SERVER_ADDR         server address, default: 0.0.0.0
  -p SERVER_PORT         server port, default: 8388
  -k PASSWORD            password
  -m METHOD              encryption method, default: aes-256-cfb
  -t TIMEOUT             timeout in seconds, default: 300
  --fast-open            use TCP_FASTOPEN, requires Linux 3.7+
  --workers WORKERS      number of workers, available on Unix/Linux
  --forbidden-ip IPLIST  comma seperated IP list forbidden to connect
  --manager-address ADDR optional server manager UDP address, see wiki

General options:
  -h, --help             show this help message and exit
  -d start/stop/restart  daemon mode
  --pid-file PID_FILE    pid file for daemon mode
  --log-file LOG_FILE    log file for daemon mode
  --user USER            username to run as
  -v, -vv                verbose mode
  -q, -qq                quiet mode, only show warnings/errors
  --version              show version information

Online help: <https://github.com/shadowsocks/shadowsocks>
```

which ssserver

```sh
ubuntu@ip-172-31-16-20:~$ which ssserver
/usr/local/bin/ssserver
```

## 启动shadowsocks服务

### 配置文件启动

````sh
sudo mkdir /etc/shadowsocks

sudo vim /etc/shadowsocks/config.json
{
	"server":"0.0.0.0",
	"server_port":1194,
	"local_address":"127.0.0.1",
	"local_port":1080,
	"password":"123qwe$%^",
	"timeout":300,
	"method":"aes-256-cfb",
	"fast_open":false,
	"workers": 1
}
````

启动：

```sh
sudo ssserver -c /etc/shadowsocks/config.json -d start
netstat -tunlp
netstat -anp  ##显示系统端口使用情况
```

停止：

```sh
sudo ssserver -c /etc/shadowsocks/config.json -d stop
```

### 命令启动

 ```sh
sudo ssserver -p 443 -k password -m rc4-md5
 ```

 如果要后台运行：

```sh
sudo ssserver -p 443 -k password -m rc4-md5 --user nobody -d start
```

如果要停止：

```sh
sudo ssserver -d stop
```

如果要检查日志：

```sh
sudo less /var/log/shadowsocks.log
```

### Docker启动

#### 安装Docker

参考官网链接：
[https://docs.docker.com/install/linux/docker-ce/ubuntu/](https://docs.docker.com/install/linux/docker-ce/ubuntu/ "链接") 

#### 下载shadowsocks镜像

```sh
docker pull oddrationale/docker-shadowsocks
```

#### 运行设置shadowsocks

```sh
docker run --rm -d -p 8388:8388 oddrationale/docker-shadowsocks -s 0.0.0.0 -p 8388 -k 'paaassswwword' -m aes-256-cfb
```

运行： `docker ps`

查看shadowsocks是否运行起来了，没问题的话就可以exit退出vps的登录了

停止容器：

```sh
docker stop --time=20 container_name
```

`8388` 服务器端的端口号，`paaassswwword` 是密码， `aes-256-cfb` 是加密方式

### 配置aws ec2安全组规则

在aws ec2控制平台添加安全组规则-入站规则

自定义TCP规则，端口，任何位置

![aws配置](/img/aws入站规则.png)

## 连接shadowsocks服务

shadowsocks服务器搭建完毕后，我们现在来客户端连接shadowsocks服务器。

shadowsocks客户端有Windows版本和Linux版本

### Windows版本

windows版本，我们可以从如下网址进行下载，如下：

https://github.com/shadowsocks/shadowsocks-windows/releases

下载完毕后，双击Shadowsocks.exe，在弹出框填入
Shadowsocks服务器的IP、Shadowsocks服务器端口，密码。

![shadowsocks客户端配置](/img/shadowsocks客户端配置.png)

参考网址：

[https://yq.aliyun.com/articles/43252](https://yq.aliyun.com/articles/43252 "链接") 

[https://yq.aliyun.com/articles/555577](https://yq.aliyun.com/articles/555577 "链接") 
