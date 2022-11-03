---
title: Docker-Compose使用自定义的网段
date: 2022-10-13 10:54:53
tags:
  - Docker Compose
  - Docker
categories:
  - Docker
---

## 默认网络

`docker-compose.yml`中创建自定义网络的时候，默认的网段为`172.*.*.*`；

使用自定义networks定义网络，docker-compose 会随机产生子网：

```yml
version: "3"

services:
  hangfire:
    image: hangfire:0.0.10
    restart: always
    hostname: hangfire
    container_name: hangfire
    ports:
      - "8100:80"
    networks:
      - net0
    volumes:
      - /etc/localtime:/etc/localtime
    environment:
      - adminConsole=true
    env_file:
      - ./var.env

networks:
  net0:
```

<!--more-->
查看docker网络：

```sh
# docker network ls
NETWORK ID     NAME                DRIVER    SCOPE
7922dab0a99d   schedule_net0       bridge    local
```

查看网络详情：

`docker network inspect schedule_net0`

```json
[
    {
        "Name": "schedule_net0",
        "Id": "7922dab0a99d1875d5a68819ff7e98bc3eb9fa2b05b490f394d256e802b9e460",
        "Created": "2022-10-13T02:43:48.774186097Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "192.168.32.0/20",
                    "Gateway": "192.168.32.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0407bdbdb75dcc73c2c6faf9bcaafc78a81dac80f454fa37f2a14ec468044705": {
                "Name": "hangfire",
                "EndpointID": "d8bae459660dfbcf2225f0a065131fa65e46ad619bdadae338c046364f8d8db3",
                "MacAddress": "02:42:c0:a8:20:02",
                "IPv4Address": "192.168.32.2/20",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "net0",
            "com.docker.compose.project": "schedule",
            "com.docker.compose.version": "2.6.1"
        }
    }
]
```

查看网关：`ifconfig`

```sh
br-7922dab0a99d: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.32.1  netmask 255.255.240.0  broadcast 192.168.47.255
        inet6 fe80::42:64ff:fe70:e248  prefixlen 64  scopeid 0x20<link>
        ether 02:42:64:70:e2:48  txqueuelen 0  (Ethernet)
        RX packets 37172  bytes 44463865 (44.4 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 28390  bytes 14401911 (14.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## 自定义网络

```yml
version: "3"

services:
  hangfire:
    image: hangfire:0.0.10
    restart: always
    hostname: hangfire
    container_name: hangfire
    ports:
      - "8100:80"
    networks:
      - net0
    volumes:
      - /etc/localtime:/etc/localtime
    environment:
      - adminConsole=true
    env_file:
      - ./var.env
networks:
  net0:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 10.10.12.0/24
        gateway: 10.10.12.1
```

查看网络配置：

`docker network inspect schedule_net0`:

```json
[
    {
        "Name": "schedule_net0",
        "Id": "1d0840de2e9c6730bcac0d70c7d0a89380624acf90f6cc4a53e441afe80c28d2",
        "Created": "2022-10-13T03:19:19.439703109Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "10.10.12.0/24",
                    "Gateway": "10.10.12.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "50e869cb7f30cd5590f6bb61ba3b02cf5cb25c5bd6204319d370a510cad76054": {
                "Name": "hangfire",
                "EndpointID": "b6f44df02514b7dc075d0049c06616beab92eb2c5320d39d9473fe639c8827b3",
                "MacAddress": "02:42:0a:0a:0c:02",
                "IPv4Address": "10.10.12.2/24",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {
            "com.docker.compose.network": "net0",
            "com.docker.compose.project": "schedule",
            "com.docker.compose.version": "2.6.1"
        }
    }
]
```

## 使用外部网络

创建网络：

```sh
docker network create --driver bridge --subnet 10.10.13.0/24 --gateway 10.10.13.1 docker_compose_net
```

```yml
version: "3"

networks:
  default:
    external:
      name: docker_compose_net

services:
  hangfire:
    image: hangfire:0.0.10
    restart: always
    hostname: hangfire
    container_name: hangfire
    ports:
      - "8100:80"
    networks:
      - net0
    volumes:
      - /etc/localtime:/etc/localtime
    environment:
      - adminConsole=true
    env_file:
      - ./var.env
```

参考：

[docker-compose up使用自定义的网段的两种方式（从其根源指定）](https://www.cnblogs.com/lemon-le/p/10531449.html)