---
title: win10配置java环境变量
date: 2020-07-16 15:15:37
tags:
- Java
- 环境变量
- JDK
categories: 
- Java
---

## 打开环境变量窗口

右键 `This PC`(此电脑) -> `Properties`（属性） -> `Advanced system settings`（高级系统设置） -> `Environment Variables`（环境变量）

![微信截图_20200716151758.png](/img/微信截图_20200716151758.png)

![微信截图_20200716151834.png](/img/微信截图_20200716151834.png)

![微信截图_20200716151910.png](/img/微信截图_20200716151910.png)

## 新建JAVA_HOME 变量

```java
变量名：JAVA_HOME
变量值：电脑上JDK安装的绝对路径
```

![微信截图_20200716152023.png](/img/微信截图_20200716152023.png)

## 新建/修改 CLASSPATH 变量

如果存在 `CLASSPATH` 变量，选中点击 Edit(编辑)。

如果没有，点击 New（新建）... 新建。

输入/在已有的变量值后面添加：

```java
变量名：CLASSPATH
变量值：.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;
```

![微信截图_20200716152137.png](/img/微信截图_20200716152137.png)

## 修改Path 变量

由于 win10 的不同，当选中 Path 变量的时候，系统会很方便的把所有不同路径都分开了，不会像 win7 或者 win8 那样连在一起。

![微信截图_20200716152234.png](/img/微信截图_20200716152234.png)

新建两条路径：

```java
%JAVA_HOME%\bin
%JAVA_HOME%\jre\bin
```

## 验证

```sh
java
java -version
javac
```

参考：

[Windows 10 配置Java 环境变量](https://www.runoob.com/w3cnote/windows10-java-setup.html)
