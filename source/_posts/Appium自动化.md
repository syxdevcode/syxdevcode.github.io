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

