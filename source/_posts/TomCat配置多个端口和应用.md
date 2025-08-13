---
title: TomCat配置多个端口和应用
date: 2020-09-17 13:50:43
tags:
- Linux
- CentOS7
- Java
- Tomcat
categories: 
- Tomcat
---

## 一个应用服务（service）中，配置多个端口号

一个service配置多个端口，项目可以通过多个端口访问。

修改 `vim /usr/local/apache-tomcat-8.5.40/conf/server.xml`，在Service下配置多个 `<Connector>` 即可。

```xml
<Service name="Catalina"> 
    <Connector connectionTimeout="20000" port="8013" protocol="HTTP/1.1" redirectPort="8443"/>
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>

    <Connector port="8014" protocol="HTTP/1.1" maxThreads="150" minSpareThreads="25" maxSpareThreads="75"
    enableLookups="false" redirectPort="8443" acceptCount="100"
    debug="0" connectionTimeout="20000" URIEncoding="utf-8"
    disableUploadTimeout="true" />

    <Engine defaultHost="localhost" name="Catalina"> 
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/> 
        <Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
            <Context path="/" reloadable="true" docBase="/test/webapp"></Context>
            <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
                prefix="localhost_access_log" suffix=".txt"
                pattern="%h %l %u %t &quot;%r&quot; %s %b" />
        </Host>
    </Engine>
</Service>
```

在这个应用里，可以用8080端口号访问服务，也可以用8099端口号来访问服务; 服务放置的路径由host决定，上例中服务放在 `webapps` 下。
以下两种方式访问同一个项目：

```
http://localhost:8013/项目名称
http://localhost:8014/项目名称
```

## 在一个Tomcat下配置多个服务，用不同的端口号。

配置多个service，每个service可以配置多个端口。

修改 `vim /usr/local/apache-tomcat-8.5.40/conf/server.xml` ，添加多个 `Service` 即可。

注意 `Service name`、`Engine name`、`appBase`，`端口号`。

```xml
<Service name="Catalina">
    <Connector connectionTimeout="20000" port="8013" protocol="HTTP/1.1" redirectPort="8443"/>
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
    <Engine defaultHost="localhost" name="Catalina">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
        <Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
        </Host>
    </Engine>
</Service>

<Service name="Catalina1">
    <Connector connectionTimeout="20000" port="8014" protocol="HTTP/1.1" redirectPort="8443"/>
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
    <Engine defaultHost="localhost" name="Catalina1">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
        <Host appBase="webapps1" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
    </Host>
    </Engine>
</Service>

<Service name="Catalina2">
    <Connector connectionTimeout="20000" port="8015" protocol="HTTP/1.1" redirectPort="8443"/>
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443"/>
    <Engine defaultHost="localhost" name="Catalina2">
        <Realm className="org.apache.catalina.realm.UserDatabaseRealm" resourceName="UserDatabase"/>
        <Host appBase="webapps2" autoDeploy="true" name="localhost" unpackWARs="true" xmlNamespaceAware="false" xmlValidation="false">
        </Host>
    </Engine>
</Service>
```

以上三个service，发布的路径不同，项目分别发布在 webapps、webapps1、webapps2下，

访问不同的项目的方法：

```
http://localhost:8013/项目名称1
http://localhost:8014/项目名称2
http://localhost:8015/项目名称3
```

参考：

[Tomcat 配置多个端口号或多个应用](https://blog.csdn.net/u012972294/article/details/86536484)