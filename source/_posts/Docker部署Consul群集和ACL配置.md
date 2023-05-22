---
title: Docker部署Consul群集和ACL配置
date: 2023-05-22 15:41:56
tags:
  - Docker
  - Consul
  - 分布式
categories: 
  - Consul
---

## 快速启动

### 创建目录

```sh
mkdir -p /opt/consul/consul1/config
mkdir -p /opt/consul/consul1/data
mkdir -p /opt/consul/consul1/log

mkdir -p /opt/consul/consul2/config
mkdir -p /opt/consul/consul2/data
mkdir -p /opt/consul/consul2/log

mkdir -p /opt/consul/consul3/config
mkdir -p /opt/consul/consul3/data
mkdir -p /opt/consul/consul3/log

mkdir -p /opt/consul/consul4/config
mkdir -p /opt/consul/consul4/data
mkdir -p /opt/consul/consul4/log

# 授权
chmod 777 /opt/consul/consul1/config /opt/consul/consul1/data /opt/consul/consul1/log /opt/consul/consul2/config /opt/consul/consul2/data /opt/consul/consul2/log /opt/consul/consul3/config /opt/consul/consul3/data /opt/consul/consul3/log /opt/consul/consul4/config /opt/consul/consul4/data /opt/consul/consul4/log
```

### 配置yml文件

```yml
version: '3.5'

networks:
  default:
    external:
      name: docker_compose_net

services:
  consul1:
    image: consul:latest
    container_name: consul1
    restart: unless-stopped
    command: agent -server -client=0.0.0.0 -bootstrap-expect=3 -node=consul1
    volumes:
      - /opt/consul/consul1/data:/consul/data
      - /opt/consul/consul1/config:/consul/config
      - /opt/consul/consul1/log:/consul/log
  consul2:
    image: consul:latest
    container_name: consul2
    restart: unless-stopped
    command: agent -server -client=0.0.0.0 -retry-join=consul1 -node=consul2
    volumes:
      - /opt/consul/consul2/data:/consul/data
      - /opt/consul/consul2/config:/consul/config
      - /opt/consul/consul2/log:/consul/log
  consul3:
    image: consul:latest
    container_name: consul3
    restart: unless-stopped
    command: agent -server -client=0.0.0.0 -retry-join=consul1 -node=consul3
    volumes:
      - /opt/consul/consul3/data:/consul/data
      - /opt/consul/consul3/config:/consul/config
      - /opt/consul/consul3/log:/consul/log
  consul4:
    image: consul:latest
    container_name: consul4
    restart: unless-stopped
    ports:
      - 8500:8500
    command: agent -client=0.0.0.0 -retry-join=consul1 -ui -node=client1
    volumes:
      - /opt/consul/consul4/data:/consul/data
      - /opt/consul/consul4/config:/consul/config
      - /opt/consul/consul4/log:/consul/log
```

<!--more-->
### 启动

```sh
# 启动
docker-compose -f /opt/consul/docker-compose.yml up -d

# 移除
docker-compose -f /opt/consul/docker-compose.yml down -v
```

## ACL认证

### 生成通讯密钥

```sh
# 运行一个consul实例
docker run -it --rm -d -p 8600:8500 --name=consul  consul:latest agent -server -bootstrap -ui -node=1 -client='0.0.0.0' 

docker exec consul consul keygen
FR4lbrVWs0qhsfDyWzDoFXJUCdFyegJlKLsslwkOw/w=
```

### 配置

创建目录：

```sh
mkdir -p /opt/consul/consul1/config
mkdir -p /opt/consul/consul1/data
mkdir -p /opt/consul/consul1/log

mkdir -p /opt/consul/consul2/config
mkdir -p /opt/consul/consul2/data
mkdir -p /opt/consul/consul2/log

mkdir -p /opt/consul/consul3/config
mkdir -p /opt/consul/consul3/data
mkdir -p /opt/consul/consul3/log

mkdir -p /opt/consul/consul4/config
mkdir -p /opt/consul/consul4/data
mkdir -p /opt/consul/consul4/log

# 授权
chmod 777 /opt/consul/consul1/config /opt/consul/consul1/data /opt/consul/consul1/log /opt/consul/consul2/config /opt/consul/consul2/data /opt/consul/consul2/log /opt/consul/consul3/config /opt/consul/consul3/data /opt/consul/consul3/log /opt/consul/consul4/config /opt/consul/consul4/data /opt/consul/consul4/log
```

### 节点配置文件

节点1：作为Leader（管理）节点，服务节点

```sh
vim /opt/consul/consul1/config/config.json
```

```json
{
    "datacenter": "dc1",
    "bootstrap_expect": 3,
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",
    "node_name": "consul1",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": false,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    },
    "encrypt": "FR4lbrVWs0qhsfDyWzDoFXJUCdFyegJlKLsslwkOw/w="
}
```

节点2：服务节点

```sh
vim /opt/consul/consul2/config/config.json
```

```json
{
    "datacenter": "dc1",
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",
    "node_name": "consul2",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": false,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    },
    "encrypt": "FR4lbrVWs0qhsfDyWzDoFXJUCdFyegJlKLsslwkOw/w="
}
```

节点3：服务节点

```sh
vim /opt/consul/consul3/config/config.json
```

```json
{
    "datacenter": "dc1",
    "data_dir": "/consul/data",
    "log_file": "/consul/log/",
    "log_level": "INFO",
    "node_name": "consul3",
    "client_addr": "0.0.0.0",
    "server": true,
    "ui": false,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    },
    "encrypt": "FR4lbrVWs0qhsfDyWzDoFXJUCdFyegJlKLsslwkOw/w="
}
```

节点4：作为客户端，启用UI

```sh
vim /opt/consul/consul4/config/config.json
```

```json
{
    "datacenter": "dc1",
    "data_dir": "/consul/data",
    "retry_join": ["consul1"],
    "log_file": "/consul/log/",
    "log_level": "INFO",
    "node_name": "client1",
    "client_addr": "0.0.0.0",
    "server": false,
    "ui": true,
    "enable_script_checks": true,
    "addresses": {
        "https": "0.0.0.0",
        "dns": "0.0.0.0"
    },
    "encrypt": "FR4lbrVWs0qhsfDyWzDoFXJUCdFyegJlKLsslwkOw/w="
}
```

### ACL token权限配置

使用linux的 `uuidgen` 命令生成一个64位UUID作为 `Master Token`：

```sh
uuidgen
a4f64bd5-100c-452d-81bd-326687d5fc80
```

如果提示 `-bash: uuidgen: command not found`:
需要安装 uuid：

```sh
apt-get install -y uuid-runtime
```

### 配置文件

```yml
version: '3.5'

networks:
  default:
    external:
      name: docker_compose_net

services:
  consul1:
    image: consul:latest
    container_name: consul1
    restart: unless-stopped
    command: agent
    volumes:
      - /opt/consul/consul1/data:/consul/data
      - /opt/consul/consul1/config:/consul/config
      - /opt/consul/consul1/log:/consul/log
  consul2:
    image: consul:latest
    container_name: consul2
    restart: unless-stopped
    command: agent -retry-join=consul1
    volumes:
      - /opt/consul/consul2/data:/consul/data
      - /opt/consul/consul2/config:/consul/config
      - /opt/consul/consul2/log:/consul/log
  consul3:
    image: consul:latest
    container_name: consul3
    restart: unless-stopped
    command: agent -retry-join=consul1
    volumes:
      - /opt/consul/consul3/data:/consul/data
      - /opt/consul/consul3/config:/consul/config
      - /opt/consul/consul3/log:/consul/log
  consul4:
    image: consul:latest
    container_name: consul4
    restart: unless-stopped
    command: agent
    ports:
      - 8500:8500
    volumes:
      - /opt/consul/consul4/data:/consul/data
      - /opt/consul/consul4/config:/consul/config
      - /opt/consul/consul4/log:/consul/log
```

```sh
vim /opt/consul/consul1/config/acl.hcl
vim /opt/consul/consul2/config/acl.hcl
vim /opt/consul/consul3/config/acl.hcl
vim /opt/consul/consul4/config/acl.hcl
```

```json
primary_datacenter = "dc1"
acl {
  enabled = true
  default_policy = "deny"
  enable_token_persistence = true
  tokens {
    master = "a4f64bd5-100c-452d-81bd-326687d5fc80"
  }
}
```

```sh
# 启动
docker-compose -f /opt/consul/docker-compose.yml up -d

# 移除
docker-compose -f /opt/consul/docker-compose.yml down -v

# 更新
docker-compose -f /opt/consul/docker-compose.yml up -d --build
```

参考：

[Docker下部署Consul集群和ACL权限配置](https://www.codenong.com/cs105951195/)

[Agents Configuration File Reference](https://developer.hashicorp.com/consul/docs/agent/config/config-files)

[Agents Command-line Reference](https://developer.hashicorp.com/consul/docs/agent/config/cli-flags)
