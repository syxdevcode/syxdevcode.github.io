---
title: UniApp-H5页面部署到IIS
date: 2021-08-18 18:18:58
tags:
- UniApp
- H5
- Vue
- IIS
categories:
- UniApp
---

## IIS 安装

下载 
[urlrewrite2.exe](https://www.iis.net/downloads/microsoft/url-rewrite)

## 配置web.config

```cs
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <system.webServer>
    <rewrite>
      <rules>
        <rule name="Handle History Mode and custom 404/500" stopProcessing="true">
          <match url="(.*)" />
          <conditions logicalGrouping="MatchAll">
            <add input="{REQUEST_FILENAME}" matchType="IsFile" negate="true" />
            <add input="{REQUEST_FILENAME}" matchType="IsDirectory" negate="true" />
          </conditions>
          <action type="Rewrite" url="/index.html" />
        </rule>
      </rules>
    </rewrite>
  </system.webServer>
</configuration>
```

## 部署

发布：

![微信截图_20210818182445.png](/img/微信截图_20210818182445.png)

目录结构：

![微信截图_20210818182611.png](/img/微信截图_20210818182611.png)