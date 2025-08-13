---
title: Tomcat支持IPV4端口
date: 2020-08-31 14:11:46
tags:
- Linux
- CentOS7
- Java
- Tomcat
categories: 
- Tomcat
---

## 支持IPV4端口

`/conf/context.xml`

```xml
<Connector port="80" maxHttpHeaderSize="8192" 
            maxThreads="300" minSpareThreads="25" maxSpareThreads="75"
            enableLookups="false" redirectPort="8443" acceptCount="100"
            connectionTimeout="20000" disableUploadTimeout="true" />
```

添加： `address="0.0.0.0"`

```xml
<Connector port="80" maxHttpHeaderSize="8192" address="0.0.0.0"
            maxThreads="300" minSpareThreads="25" maxSpareThreads="75"
            enableLookups="false" redirectPort="8443" acceptCount="100"
            connectionTimeout="20000" disableUploadTimeout="true" />
```

## 只支持IP4

### linux平台

在 `<tomcat>/bin` 目录下 `catalina.sh`，添加如下内容：

`vim catalina.sh`，然后输入 `/` 搜索 `JAVA_OPTS`,

`n` 向下查找
`N` 向上查找

![微信截图_20200831141600.png](/img/微信截图_20200831141600.png)

```sh
# 添加以下内容
JAVA_OPTS="$JAVA_OPTS -Djava.net.preferIPv4Stack=true -Djava.net.preferIPv4Addresses=true "
```

### windows

编辑 bin 目录下的 `catalina.bat` 文件，改成以下内容：

```sh
set "JAVA_OPTS=%JAVA_OPTS% -Djava.net.preferIPv4Stack=true %JSSE_OPTS%"
```

参考

[如何让tomcat只支持ipv4](https://blog.csdn.net/weiqubo/article/details/49820101)
