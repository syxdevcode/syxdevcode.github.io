---
title: Docker安装Portainer
date: 2023-05-22 14:38:48
tags:
  - Ubuntu
  - Linux
  - Docker
  - Docker Compose
categories:
  - Docker
---

```sh
docker run -d --restart=always --name="portainer" -p 9200:9000 -v /var/run/docker.sock:/var/run/docker.sock 6053537/portainer-ce
```

在浏览器打开 Host:9200，首次进入按照要求设置用户名，密码。
