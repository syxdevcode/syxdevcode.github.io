---
title: Appium自动化
date: 2020-12-23 10:28:34
tags:
- Appium
- CSharp
- Java
- Android
- 自动化
categories: Appium
---

## 简介

Appium 是一个开源的自动化测试工具，支持 Android、iOS 平台上的原生应用，支持 Java、Python、PHP 等多种语言。

Appium 封装了 Selenium，能够为用户提供所有常见的 JSON 格式的 Selenium 命令以及额外的移动设备相关的控制命令，比如：多点触控手势、屏幕朝向等。

## 配置环境

### Java环境

右键 This PC(此电脑) -> Properties（属性） -> Advanced system settings（高级系统设置） -> Environment Variables（环境变量）

![微信截图_20201223105939.png](/img/微信截图_20201223105939.png)

新建/修改 CLASSPATH 变量

![微信截图_20201223110109.png](/img/微信截图_20201223110109.png)

Path 变量末尾添加 ;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

![微信截图_20201223110433.png](/img/微信截图_20201223110433.png)

### Android-sdk

新建环境变量 `ANDROID_HOME`，变量值为 `android-sdk` 位置，
如：`E:\BaiduNetdiskDownload\appium\android-sdk\android-sdk-windows`

在 Path 变量值的末尾添加:
`;%ANDROID_HOME%\tools;%ANDROID_HOME%\build-tools\30.0.0-preview;%ANDROID_HOME%\platform-tools`

![微信截图_20201223111155.png](/img/微信截图_20201223111155.png)

### mumu模拟器

通过安卓的 adb 连接虚拟机，需要在控制台执行 adb connect 127.0.0.1:7555 命令，让 adb 连接上虚拟机，在 cmd 控制台输入 adb devices 查看当前连接的虚拟机。

### Android真机

**开启开发者模式**

荣耀V9手机，荣耀V10、荣耀V20、华为P20、华为P30等型号：

1，进入手机的设置项 `设置 - 关于手机` 。
2，点击 `版本号` 7次进入开发者模式的，选择 `返回` 就可以看到 `开发者选项` 的菜单。

进入开发者选项 -> 开启开发者选项，并分别开启：USB调试、USB安装、USB调试（安全设置）选项，USB连接时，选择 `文件传输`。

如果手机是第一次链接电脑可能还需要安装一下驱动: Android手机驱动。

## 使用

1，启动 Appium 和 mumu（如果使用Android真机，可以忽略开启mumu）

![微信截图_20210104135431.png](/img/微信截图_20210104135431.png)

2，





参考：

[Python + Appium 自动化操作微信入门看这一篇就够了](https://blog.csdn.net/ityard/article/details/109498443)
