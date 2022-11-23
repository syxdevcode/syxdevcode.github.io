---
title: Prometheus监控Linux和Docker
date: 2022-11-23 15:21:10
tags:
  - Ubuntu
  - Grafana
  - Prometheus
  - Promtail
  - Docker Compose
  - Docker
categories:
  - Grafana
---

## 简介

prometheus：负责收集和存储时间序列数据
alertmanager: 警告信息通知
node-exporter：负责暴露主机 metrics 数据给 prometheus
cadvisor： 收集docker容器信息
grafana：负责展示数据

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
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['10.10.0.106:9993']
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

```yml
groups:
- name: node_down
  rules:
  - alert: InstanceDown
    expr: up == 0
    for: 1m
    labels:
      user: test
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 1 minutes."
```

`alertmanager.yml`文件内容:

```yml
global: 
  smtp_smarthost: 'smtp.xxx.com:25'
  smtp_from: 'user@xxx.com'
  smtp_auth_username: 'user@xxx.com'
  smtp_auth_password: 'pwd123456'
  smtp_require_tls: false

route: 
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 10m
  receiver: live-monitoring

receivers: 
  - name: 'live-monitoring'
    email_configs: 
    - to: 'xxx@qq.com'
```

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

```sh
# 启动
docker-compose up -d

# 移除
docker-compose down -v
```

## 配置Grafana

1，配置数据源

![Snipaste_2022-11-23_18-02-31.png](/img1/Snipaste_2022-11-23_18-02-31.png)

![Snipaste_2022-11-23_18-04-08.png](/img1/Snipaste_2022-11-23_18-04-08.png)

![Snipaste_2022-11-23_18-03-12.png](/img1/Snipaste_2022-11-23_18-03-12.png)

2，导入仪表板模板

官方仪表板地址：[https://grafana.com/grafana/dashboards/](https://grafana.com/grafana/dashboards/)

![Snipaste_2022-11-23_18-08-37.png](/img1/Snipaste_2022-11-23_18-08-37.png)

输入模板：893,9852,15172,1860 等等。

![Snipaste_2022-11-23_18-11-03.png](/img1/Snipaste_2022-11-23_18-11-03.png)

参考：

[使用docker-compose快速搭建prometheus](https://www.jianshu.com/p/689610513a62)