---
title: MinIo-Windows离线安装
date: 2022-03-21 15:35:41
tags:
- Windows
- MinIo
- 安装部署
categories:
- MinIo
---

## winsw安装Windows服务

1，下载安装：

[https://github.com/winsw/winsw](https://github.com/winsw/winsw)

2，将WinSW.exe复制到自定义的目录，并重命名为自己想命名的服务名称minio-server.exe。

3，同目录下创建minio-server.xml。特别注意，xml和exe必须同名

4，配置minio-server.xml文件
<!--more-->
minio-server.xml文件：

```xml
<service>
    <id>minio-server</id>
    <name>minio-server</name>
    <description>minio文件存储服务器</description>
    <!-- 可设置环境变量 -->
    <env name="HOME" value="%BASE%"/>
    <env name="MINIO_ROOT_USER" value="admin123456"/>
    <!-- 密码不能少于8位，否则报错 -->
    <env name="MINIO_ROOT_PASSWORD" value="admin123456"/>
    <executable>%BASE%\minio.exe</executable>
    <arguments>server "%BASE%\data" --console-address ":9001"</arguments>
    <!-- <logmode>rotate</logmode> -->
    <logpath>%BASE%\logs</logpath>
    <log mode="roll-by-size-time">
      <sizeThreshold>10240</sizeThreshold>
      <pattern>yyyyMMdd</pattern>
      <autoRollAtTime>00:00:00</autoRollAtTime>
      <zipOlderThanNumDays>5</zipOlderThanNumDays>
      <zipDateFormat>yyyyMMdd</zipDateFormat>
    </log>
</service>
```

5，安装服务

```ps1
// 安装
D:\MinIO>minio-server install
Installing service 'minio-server (minio-server)'...
Service 'minio-server (minio-server)' was installed successfully.

// 卸载 需要停止服务
D:\MinIO>minio-server uninstall
Uninstalling service 'minio-server (minio-server)'...
Service 'minio-server (minio-server)' was uninstalled successfully.
```

打开服务，启动 `minio-server`。

目录结构如下：

![Snipaste_2022-03-21_19-53-09.png](/img/Snipaste_2022-03-21_19-53-09.png)

## 验证安装

打开：`http://localhost:9001`

![Snipaste_2022-03-21_19-55-26.png](/img/Snipaste_2022-03-21_19-55-26.png)

参考：

[https://www.cnblogs.com/zys-blog/p/13164197.html](https://www.cnblogs.com/zys-blog/p/13164197.html)