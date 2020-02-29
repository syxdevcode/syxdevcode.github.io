---
title: Centos7安装Docker-Compose
date: 2018-08-02 10:32:55
tags:
- Linux
- CentOS7
- Docker Compose
- Docker
- 安装部署
categories: 
- Docker Compose
---

# 下载包

```docker
sudo curl -L https://github.com/docker/compose/releases/download/1.22.0/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
```

注意出现：`curl: (35) Peer reports incompatible or unsupported protocol version.`

需要更新nss,curl

```bash
sudo yum update nss curl
```

# 授权

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

# 查看版本

```bash
docker-compose --version
```

参考：

[https://docs.docker.com/compose/install/#install-compose](https://docs.docker.com/compose/install/#install-compose)