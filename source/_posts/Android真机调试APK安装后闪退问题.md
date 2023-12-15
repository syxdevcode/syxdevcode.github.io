---
title: Android真机调试APK安装后闪退问题
date: 2023-10-12 14:02:45
tags:
  - MAUI
  - Android
categories:
  - MAUI
---

GitBash 的 shell 命令行工具，可以试用grep命令过滤：

注意，需要把 adb.exe 添加到环境变量中，如下：

![Snipaste_2023-10-12_14-16-36.png](/img1/Snipaste_2023-10-12_14-16-36.png)

```sh
# 产设备列表
adb.exe devices

# 显示手机已安装应用目录
# -s 设备名
adb.exe -s 340782513600AP8 shell pm list packages -3

#  过滤com.test.masablazorapp包日志
adb.exe -s 340782513600AP8 logcat | grep com.test.masablazorapp

# 把日志保存到指定目录
adb.exe -s 340782513600AP8 logcat -v time > e:\log.txt *:E
```
<!--more-->
日志级别：

```sh
V：详细（最低优先级）
D：除错
I：信息
W：警告
E：错误
F：严重
S：静默（最高优先级，未曾输出过任何内容）
```

日志示例如下：

```sh
10-12 09:01:30.727  6714  6714 V GraphicsEnvironment: ANGLE Developer option for 'com.test.masablazorapp' set to: 'default'
10-12 09:01:30.727  6714  6714 V GraphicsEnvironment: ANGLE GameManagerService for com.test.masablazorapp: false
10-12 09:01:31.071  6756  6756 F DEBUG   : Cmdline: com.test.masablazorapp
10-12 09:01:31.071  6756  6756 F DEBUG   : pid: 6714, tid: 6714, name: y.masablazorapp  >>> com.test.masablazorapp <<<
10-12 09:01:31.071  6756  6756 F DEBUG   :       #01 pc 0000000000039f1c  /data/app/~~FK2zjiAFbo4PYevfAQsr1Q==/com.test.masablazorapp-25GJnPc9BdbEbWkerGRRlQ==/lib/arm64/libmonodroid.so (xamarin::android::internal::MonodroidRuntime::mono_log_handler(char const*, char const*, char const*, int, void*)+144) (BuildId: a8466986ce0a58d7eb3a452ef823ee9785ed332d)
10-12 09:01:31.071  6756  6756 F DEBUG   :       #02 pc 00000000002da9a8  /data/app/~~FK2zjiAFbo4PYevfAQsr1Q==/com.test.masablazorapp-25GJnPc9BdbEbWkerGRRlQ==/lib/arm64/libmonosgen-2.0.so (BuildId: 0deb0abafacd23fba0e668d911869b6446ca3009)
10-12 09:01:31.071  6756  6756 F DEBUG   :       #03 pc 00000000002daad4  /data/app/~~FK2zjiAFbo4PYevfAQsr1Q==/com.test.masablazorapp-25GJnPc9BdbEbWkerGRRlQ==/lib/arm64/libmonosgen-2.0.so (BuildId: 0deb0abafacd23fba0e668d911869b6446ca3009)
10-12 09:01:31.071  6756  6756 F DEBUG   :       #04 pc 00000000002548f8  /data/app/~~FK2zjiAFbo4PYevfAQsr1Q==/com.test.masablazorapp-25GJnPc9BdbEbWkerGRRlQ==/lib/arm64/libmonosgen-2.0.so (BuildId: 0deb0abafacd23fba0e668d911869b6446ca3009)
10-12 09:01:31.071  6756  6756 F DEBUG   :       #05 pc 0000000000252e6c  /data/app/~~FK2zjiAFbo4PYevfAQsr1Q==/com.test.masablazorapp-25GJnPc9BdbEbWkerGRRlQ==/lib/arm64/libmonosgen-2.0.so (BuildId: 0deb0abafacd23fba0e668d911869b6446ca3009)
10-12 09:01:31.071  6756  6756 F DEBUG   :       #06 pc 0000000000265b44  /data/app/~~FK2zjiAFbo4PYevfAQsr1Q==/com.test.masablazorapp-25GJnPc9BdbEbWkerGRRlQ==/lib/arm64/libmonosgen-2.0.so (BuildId: 0deb0abafacd23fba0e668d911869b6446ca3009)
```

参考：

[MAUI APK安装到其他手机闪退问题](https://www.cnblogs.com/wgscd/p/16403816.html)