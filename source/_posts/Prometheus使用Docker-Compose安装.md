---
title: Prometheus监控Linux和Docker
date: 2022-11-23 15:21:10
tags:
  - Ubuntu
  - Grafana
  - Prometheus
  - Docker Compose
  - Docker
categories:
  - Grafana
---

## 简介

* prometheus：负责收集和存储时间序列数据
* alertmanager: 警告信息通知
* node-exporter：负责暴露主机 metrics 数据给 prometheus
* cadvisor： 收集docker容器信息
* grafana：负责展示数据

## Docker-Compose安装Prometheus

`docker-compose.yml` 配置：

```yml
version: '3.7'

networks:
  default:
    external:
      name: docker_compose_net

services:
    prometheus:
        image: prom/prometheus
        container_name: prometheus
        hostname: prometheus
        restart: unless-stopped
        volumes:
            - /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
            - /opt/prometheus/node_down.yml:/etc/prometheus/node_down.yml
        ports:
            - "9990:9090"
        environment:
            - TZ=Asia/Shanghai # 时区配置亚洲上海

    alertmanager:
        image: prom/alertmanager
        container_name: alertmanager
        hostname: alertmanager
        restart: unless-stopped
        volumes:
            - /opt/prometheus/alertmanager.yml:/etc/alertmanager/alertmanager.yml
            - /opt/prometheus/alert-template/:/alertmanager/template/
        ports:
            - "9993:9093"
        environment:
            - TZ=Asia/Shanghai # 时区配置亚洲上海
```
<!--more-->

`prometheus.yml`文件内容：

```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  scrape_timeout:      15s # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['10.10.0.106:9993'] # 设定alertmanager和prometheus交互的接口，即alertmanager监听的ip地址和端口
      # - alertmanager:9993

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "node_down.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'
    static_configs:
      - targets: ['10.10.0.106:9990']

  - job_name: 'cadvisor-106'
    static_configs:
      - targets: ['10.10.0.106:9980']

  - job_name: 'node-106'
    scrape_interval: 8s
    static_configs:
      - targets: ['10.10.0.106:9900']
      
  - job_name: 'cadvisor-102'
    static_configs:
      - targets: ['10.10.0.102:9980']

  - job_name: 'node-102'
    scrape_interval: 8s
    static_configs:
      - targets: ['10.10.0.102:9900']
      
  - job_name: 'cadvisor-103'
    static_configs:
      - targets: ['10.10.0.103:9980']

  - job_name: 'node-103'
    scrape_interval: 8s
    static_configs:
      - targets: ['10.10.0.103:9900']
```

`node_down.yml`文件内容：

注：注意格式，同级对齐。

```yml
groups:
  - name: 主机状态-监控告警
    rules:
    - alert: 主机失联
      expr: up == 0 # 告警的判定条件，参考Prometheus高级查询来设定
      for: 20s # 满足告警条件持续时间多久后，才会发送告警
      labels: #标签项
        status: 非常严重
      annotations: # 解析项，详细解释告警信息
        severity: "非常严重"
        summary: "{{$labels.instance}}:服务器宕机"
        description: "{{$labels.instance}}:服务器超过20秒无法连接"

    - alert: CPU使用率过高
      expr: 100-(avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) by(instance,company,ownningsystem)* 100) > 90
      for: 25s
      labels:
        status: 一般告警
      annotations:
        severity: "一般告警"
        summary: "{{$labels.mountpoint}} CPU使用率过高！"
        description: "{{$labels.mountpoint }} CPU使用大于90%(目前使用:{{humanize  $value}}%)"

    - alert: 内存使用率过高
      expr: 100 -(node_memory_MemTotal_bytes -node_memory_MemFree_bytes+node_memory_Buffers_bytes+node_memory_Cached_bytes ) / node_memory_MemTotal_bytes * 100> 85
      for: 30s
      labels:
        status: 严重告警
      annotations:
        severity: "严重告警"
        summary: "{{$labels.mountpoint}} 内存使用率过高！"
        description: "{{$labels.mountpoint }} 内存使用大于85%(目前使用:{{humanize $value}}%)"

    - alert: 磁盘分区使用率过高
      expr: 100-(node_filesystem_free_bytes{fstype=~"ext4|xfs"}/node_filesystem_size_bytes {fstype=~"ext4|xfs"}*100) > 85
      for: 20s
      labels:
        status: 严重告警
      annotations:
        severity: "严重告警"
        summary: "{{$labels.mountpoint}} 磁盘分区使用率过高！"
        description: "{{$labels.mountpoint }} 磁盘分区使用大于85%(目前使用:{{humanize $value}}%)"

    - alert: 主机重启
      expr: sum(time() - node_boot_time_seconds)by(instance,company,ownningsystem)/60/60 <= 12
      for: 1m
      labels:
        status: 非常严重
      annotations:
        severity: "严重告警"
        summary: "{{$labels.mountpoint}} 主机重启"
        description: "{{$labels.mountpoint }} 主机在过去12h内发生重启，请确认业务是否正常！(重启后已运行时间：{{humanize $value}}h)"
```

告警信息生命周期的3中状态:

* 1）inactive：表示当前报警信息即不是firing状态也不是pending状态
* 2）pending：表示在设置的阈值时间范围内被激活的
* 3）firing：表示超过设置的阈值时间被激活的

`alertmanager.yml`文件内容:

官方文档地址：[https://prometheus.io/docs/alerting/latest/configuration/](https://prometheus.io/docs/alerting/latest/configuration/)

```yml
# 全局配置项　
global: 
  resolve_timeout: 5m    #处理超时时间，默认为5min
  smtp_smarthost: 'smtp.163.com:465'  # 邮箱smtp服务器代理
  smtp_from: 'zhong@163.com'  # 发送邮箱名称
  smtp_auth_username: 'zhong@163.com'  # 邮箱名称
  smtp_auth_password: 'DI' # 邮箱密码或授权码
  smtp_require_tls: false

# 定义模板路径
templates:
  - '/alertmanager/template/*.tmpl' #重要：注意映射的路径（绝对路径），不正确会报错：no such template \"node_down.html\"

route: 
  # 这里的标签列表是接收到报警信息后的重新分组标签，例如，接收到的报警信息里面有许多具有
  # cluster=A 和 alertname=LatncyHigh 这样的标签的报警信息将会批量被聚合到一个分组里面
  group_by: ['alertname'] # 报警分组依据
  group_wait: 10s  # 最初即第一次等待多久时间发送一组警报的通知
  group_interval: 20s # 在发送新警报前的等待时间
  repeat_interval: 20m  # 发送重复警报的周期
  receiver: 'live-monitoring-email' # 发送警报的接收者的名称

  #routes:
  #- receiver: 'live-monitoring-dingtalk'
  #  matchers:
  #  - service = ~"minio|redis"

# 定义警报接收者信息
receivers: 
  - name: 'live-monitoring-email'
    email_configs: 
    - send_resolved: true
      to: '198@qq.com'
      html: '{{ template "node_down.html" . }}' # 设定邮箱的内容模板
      headers: { Subject: " 【监控告警】 {{ .CommonLabels.alertname }} "} # 接收邮件的标题
    webhook_configs: # 发送钉钉通知
    - send_resolved: true
      url: 'http://10.10.0.17:30008/Alert/Alertmanager'

  #- name: 'live-monitoring-dingtalk'
  #  webhook_configs: # 发送钉钉通知
  #  - send_resolved: true
  #    url: 'http://127.0.0.1'


# 抑制器配置
inhibit_rules: 
  - source_match: 
     severity: 'critical' 
    target_match: 
     severity: 'warning' #目标告警状态
    equal: ['alertname', 'dev', 'instance']
```

## .tmpl模板的配置

官方语法介绍：[https://prometheus.io/docs/alerting/latest/notifications/](https://prometheus.io/docs/alerting/latest/notifications/)

官方模板链接：[https://github.com/prometheus/alertmanager/blob/main/template/default.tmpl](https://github.com/prometheus/alertmanager/blob/main/template/default.tmpl)

### node_down.tmpl

注：Labels项，表示prometheus里面的可选label项。annotation项表示报警规则中定义的annotation项的内容。

```html
#########基于alertmanager官方模板修改，内容可删减#########

{{ define "__alertmanager" }}Alertmanager{{ end }}

{{ define "__subject" }}[{{ .Status | toUpper }}{{ if eq .Status "firing" }}:{{ .Alerts.Firing | len }}{{ end }}] {{ .GroupLabels.SortedPairs.Values | join " " }} {{ if gt (len .CommonLabels) (len .GroupLabels) }}({{ with .CommonLabels.Remove .GroupLabels.Names }}{{ .Values | join " " }}{{ end }}){{ end }}{{ end }}
{{ define "__description" }}{{ end }}

{{ define "node_down.html" }}
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<!--
Style and HTML derived from https://github.com/mailgun/transactional-email-templates

The MIT License (MIT)

Copyright (c) 2014 Mailgun

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
-->
<html xmlns="http://www.w3.org/1999/xhtml" xmlns="http://www.w3.org/1999/xhtml" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
<head style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
<meta name="viewport" content="width=device-width" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;" />
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;" />
<title style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">{{ template "__subject" . }}</title>

</head>

<body itemscope="" itemtype="http://schema.org/EmailMessage" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; -webkit-font-smoothing: antialiased; -webkit-text-size-adjust: none; height: 100%; line-height: 1.6em; width: 100% !important; background-color: #f6f6f6; margin: 0; padding: 0;" bgcolor="#f6f6f6">

<table style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; width: 100%; background-color: #f6f6f6; margin: 0;" bgcolor="#f6f6f6">
  <div style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; max-width: 600px; display: block; margin: 0 auto; padding: 0;">
    <table width="100%" cellpadding="0" cellspacing="0" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; border-radius: 3px; background-color: #fff; margin: 0; border: 1px solid #e9e9e9;" bgcolor="#fff">
      <tr style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
        <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: top; color: #fff; font-weight: 500; text-align: center; border-radius: 3px 3px 0 0; background-color: #E6522C; margin: 0; padding: 20px;" align="center" bgcolor="#E6522C" valign="top">
          发生 {{ .Alerts | len }}  个 {{ range .GroupLabels.SortedPairs }}
                {{ .Value }}
          {{ end }} 告警 ！！请尽快处理
        </td>
      </tr>
      <tr style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
        <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 10px;" valign="top">
          <table border="1" cellpadding="2" cellspacing="0" width="100%" cellpadding="0" cellspacing="0" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
            <tr border="1" cellpadding="2" cellspacing="0" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
            </tr>
          <table border="1" cellpadding="2" cellspacing="0" width="100%" cellpadding="0" cellspacing="0" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
            <tr style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
              <td width="50px" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: middle; margin: 0; padding: 3 3 3px;" valign="top" >
                <strong>告警名称</strong> 
              </td>
              <td width="50px" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: middle; margin: 0; padding: 3 3 3px;" valign="top" >
                <strong>告警级别</strong> 
              <td width="50px" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: middle; margin: 0; padding: 3 3 3px;" valign="top" >
                <strong>实例</strong> 
              <td width="50px" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: middle; margin: 0; padding: 3 3 3px;" valign="top" >
                <strong>所属系统</strong> 
              <td width="50px" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: middle; margin: 0; padding: 3 3 3px;" valign="top" >
                <strong>厂商</strong> 
              <td width="50px" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: middle; margin: 0; padding: 3 3 3px;" valign="top" >
                <strong>触发时间</strong>
              </td>
              <td width="50px" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 16px; vertical-align: middle; margin: 0; padding: 3 3 3px;" valign="top" >
                <strong>说明</strong> 
              </td>
            </tr>
            {{ range .Alerts.Firing }}
            <tr style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
              <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 3 3 3px;" valign="top">
               <!-- {{ .Labels.status }} -->
                {{ .Annotations.severity }}
              </td>
              <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 3 3 3px;" valign="top">
                {{ .Status }}
              </td>
              <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 3 3 3px;" valign="top">
                {{ .Labels.instance }}
              </td>
              <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 3 3 3px;" valign="top">
                {{ .Labels.ownningsystem }}
              </td>
              <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 3 3 3px;" valign="top">
                {{ .Labels.company }}
              </td>
              <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 3 3 3px;" valign="top">
                {{  .StartsAt.Format "2006-01-02 15:04:05"  }}
              </td>
              <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; vertical-align: top; margin: 0; padding: 3 3 3px;" valign="top">
                {{ .Annotations.description  }}
              </td>
            </tr>
            {{ end }}
          </table>
        </td>
      </tr>
      </table>
    </table>

    <div style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; width: 100%; clear: both; color: #999; margin: 0; padding: 20px;">
      <table width="100%" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
        <tr style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 14px; margin: 0;">
          <td style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 12px; vertical-align: top; text-align: center; color: #999; margin: 0; padding: 0 0 20px;" align="center" valign="top"><a href="{{ .ExternalURL }}" style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; box-sizing: border-box; font-size: 12px; color: #999; text-decoration: underline; margin: 0;">Sent by {{ template "__alertmanager" . }}</a></td>
        </tr>
      </table>
    </div></div>
</table>

</body>
</html>
{{ end }}
```

## 启动/销毁容器

```sh
# 启动
docker-compose up -d

# 移除
docker-compose down -v
```

## 各Linux主机配置

```yml
version: '3.7'

networks:
  default:
    external:
      name: docker_compose_net

services:
    node-exporter:
        image: quay.io/prometheus/node-exporter
        container_name: node-exporter
        hostname: node-exporter
        restart: unless-stopped
        ports:
            - "9900:9100"
        environment:
            - TZ=Asia/Shanghai # 时区配置亚洲上海

    cadvisor:
        image: zcube/cadvisor
        container_name: cadvisor
        hostname: cadvisor
        restart: unless-stopped
        volumes:
            - /:/rootfs:ro
            - /var/run:/var/run:rw
            - /sys:/sys:ro
            - /var/lib/docker/:/var/lib/docker:ro
            - /dev/disk/:/dev/disk:ro
        devices:
            - /dev/kmsg:/dev/kmsg
        ports:
            - "9980:8080"
        environment:
            - TZ=Asia/Shanghai # 时区配置亚洲上海
```

备注：Ubuntu22.04系统下，cadvisor汇报异常信息。

```sh
W1123 09:25:29.510214       1 manager.go:159] Cannot detect current cgroup on cgroup v2
W1123 09:25:29.664485       1 manager.go:288] Could not configure a source for OOM detection, disabling OOM events: open /dev/kmsg: operation not permitted
```

```sh
# 启动
docker-compose up -d

# 移除
docker-compose down -v
```

查看监控信息：

```sh
# alertmanager 信息
http://10.10.0.106:9993/#/alerts

# prometheus 信息
http://10.10.0.106:9990/targets
```

## 配置Grafana

1，配置数据源

![Snipaste_2022-11-23_18-02-31.png](/img1/Snipaste_2022-11-23_18-02-31.png)

![Snipaste_2022-11-23_18-04-08.png](/img1/Snipaste_2022-11-23_18-04-08.png)

![Snipaste_2022-11-23_18-03-12.png](/img1/Snipaste_2022-11-23_18-03-12.png)

2，导入仪表板模板

官方仪表板地址：[https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/)

![Snipaste_2022-11-23_18-08-37.png](/img1/Snipaste_2022-11-23_18-08-37.png)

输入模板：8919,893,9852,15172,1860 等等。

![Snipaste_2022-11-23_18-11-03.png](/img1/Snipaste_2022-11-23_18-11-03.png)

参考：

[使用docker-compose快速搭建prometheus](https://www.jianshu.com/p/689610513a62)

[Prometheus 和 Alertmanager实战配置](https://www.cnblogs.com/longcnblogs/p/9620733.html)

[Alertmanager自定义邮件告警模板](https://www.jianshu.com/p/264cbf8d618b)
