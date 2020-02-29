---
title: CentOS7使用Jexus部署.NetCore应用
date: 2017-05-24 13:57:28
tags:
- Linux
- CentOS7
- DotNetCore
- Jexus
categories:
- Jexus
---
# CentOS7使用Jexus部署.NetCore应用

## 安装Jexus

安装 Jexus 直接使用一下命令即可(需要在root身份下执行):

``` linux
curl https://jexus.org/release/x64/install.sh|sh
```

安装成功后会提示：OK, Jexus has been installed in /usr/jexus.

## 发布程序到CentOS7服务器

在`/usr/local`建立www文件夹，将应用程序包放在www文件夹下

如果www不存在，则使用命令创建：

``` linux
mkdir /usr/local/www
```

将发布好的程序，拷贝到linux系统/usr/local/www文件夹中

参考使用

[WinSCP配置](https://syxdevcode.github.io/2017/05/24/%E4%BD%BF%E7%94%A8WinSCP%E8%BD%AF%E4%BB%B6%E5%9C%A8windows%E5%92%8CLinux%E4%B8%AD%E8%BF%9B%E8%A1%8C%E6%96%87%E4%BB%B6%E4%BC%A0%E8%BE%93/ "WinSCP")

## 配置Jexus

```linux
cd /usr/jexus/siteconf #切换到jexus配置文件目录
cp default test #复制默认配置文件test
vim test #编辑
```

参考配置：

```linux
port=82  # 外部访问的端口号，可以改成你想要的端口号,外部访问通过 ip/域名:端口号 即可访问
roott=/ /usr/local/www/SessionTest/ # 应用程序的工作根目录(全路径)
hosts=*    #OR your.com,*.your.com  如果为服务器设置了DNS解析，则可以填写解析到服务器的域名，如:www.myweb.com

AppHost={
  cmd=dotnet SessionTest.dll; # 命令，启动Asp.Net Core应用要执行的命令
  root=/usr/local/www/SessionTest/; # Asp.Net Core应用程序所在的全路径 
  port=0;
}
```

```linux
port=0;
# Asp.Net Core应用程序所使用的端口号，如果在程序中使用了UsrUrls自定义端口则使用UsrUrls中填写的端口

(不建议使用UsrUrls自定义端口), 在没有使用UsrUrls自定义端口的情况下端口号设置为 0，Jexus会在运行时与

Asp.Net Core进行"协商"具体使用的端口号，避免多个应用分配端口的麻烦和冲突的风险。
```

反向代理

```linux
reproxy=/ http://192.168.1.132:5100,http://192.168.1.130:5100
```

## 启动/重启 Jexus

当配置文件编辑完成后使用以下命令对Jexus进行 启动/重启

```linux
# 如果已启动 Jexus：
sh /usr/jexus/jws restart

# 如果未启动 Jexus：
sh /usr/jexus/jws start
```

## 开启远程访问端口

开放jexus配置的82端口，以供外部访问：

````linux
firewall-cmd --zone=public --add-port=82/tcp --permanent
````

命令含义：

````linux
--zone #作用域

--add-port=82/tcp #添加端口，格式为：端口/通讯协议

--permanent #永久生效，没有此参数重启后失效

firewall-cmd --reload  #重启防火墙
````

参考：

[http://www.cnblogs.com/staneee/p/6852559.html](http://www.cnblogs.com/staneee/p/6852559.html "Jexus安装配置")

[https://www.jexus.org/](https://www.jexus.org/ "Jexus官网")

[https://www.linuxdot.net/bbsfile-3084](https://www.linuxdot.net/bbsfile-3084 "Jexus官网文档")