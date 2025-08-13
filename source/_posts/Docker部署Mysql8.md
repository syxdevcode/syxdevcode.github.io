---
title: Docker部署Mysql8
date: 2023-04-21 15:02:49
tags:
  - Docker Compose
  - Docker
  - Ubuntu
  - Linux
  - MySql
  - 安装部署
categories: 
  - MySql
---

## 配置

创建目录：

```sh
mkdir -p /opt/mysql8/conf
mkdir -p /opt/mysql8/db
mkdir -p /opt/mysql8/binlogs

# 授权
chmod 777 /opt/mysql8/conf
chmod 777 /opt/mysql8/db
chmod 777 /opt/mysql8/binlogs
```

目录结构如下：

```sh
├── conf
│   └── my.cnf  # MySQL配置文件
├── db # 数据库数据文件目录
├── mysql-docker-compose.yml  # docker-compose.yml文件
└── binlogs # 日志存放目录
```

<!--more-->

### docker-compose配置

输入以下命令创建 `docker-compose` 文件。

`touch /opt/mysql8/mysql-docker-compose.yml`

编辑：

`vim /opt/mysql8/mysql-docker-compose.yml`

```yml
version: '3'
services:
  mysql:
    restart: unless-stopped
    privileged: true
    image: mysql:8.0.33
    container_name: mysql8
    volumes:
      - /opt/mysql8/db:/var/lib/mysql
      - /opt/mysql8/conf:/etc/mysql/conf.d
      - /opt/mysql8/binlogs:/binlogs
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_USER=mySpider
      - MYSQL_PASSWORD=1q2w#E
      - SET_CONTAINER_TIMEZONE=true
      - MYSQL_INITDB_SKIP_TZINFO="Asia/Shanghai"
      - CONTAINER_TIMEZONE=Asia/Shanghai
      - TZ=Asia/Shanghai # 时区配置亚洲上海
    ports:
      - 13106:3306

networks:
  default:
    external:
      name: docker_compose_net
```

### mysql配置

输入以下命令创建 `my.cnf` mysql配置文件。

`touch /opt/mysql8/conf/my.cnf`

编辑：

`vim /opt/mysql8/conf/my.cnf`

```cnf
# 对其他远程连接的mysql客户端的配置
[mysql]
socket=/var/lib/mysql/mysql.sock

# 跳过授权
# skip-grant-tables

# set mysql client default chararter
default-character-set=utf8mb4

# 本地mysql服务的配置
[mysqld]
socket=/var/lib/mysql/mysql.sock
bind-address=0.0.0.0

# set mysql server port  
port = 3306 #默认是3306，防止端口冲突发生，可以避免使用 3306 mysql默认端口

# set mysql install base dir
basedir=/var/lib/mysql

# set the data store dir
datadir=/var/lib/mysql/data

# set the number of allow max connnection
# 设置置 MySQL 的最大连接，按你实际情况适当设置。出现 'Too many connections' 错误，是因为 max_connections 的值太低，需要设置更高的链接数，

# max_connection 值被设高之后的缺陷是当服务器运行超过设置阈值或更高的活动事务时会变的没有响应。
max_connections = 3000
character-set-client-handshake=FALSE

# set server charactre default encoding
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# the storage engine
default-storage-engine=INNODB
lower_case_table_names=1
max_allowed_packet=16M
explicit_defaults_for_timestamp=true

# 解决 this is incompatible with sql_mode=only_full_group_by 的错误
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION

# MySQL8 的密码认证插件
authentication_policy = mysql_native_password

# 设置时区
default-time_zone='+8:00'

# binlog 配置
log-bin = /binlogs/mysql-bin.log

max-binlog-size = 512M
binlog-format = ROW
binlog_expire_logs_seconds= 7776000 # 90天
binlog-cache-size = 4M
max-binlog-cache-size = 4096M

[mysql.server]
user=mysql
basedir=/var/lib/mysql

# 对本地的mysql客户端的配置
[client] 
default-character-set = utf8mb4
```

## 启动&移除

```sh
# 启动
docker-compose -f /opt/mysql8/mysql-docker-compose.yml up -d

# 移除
docker-compose -f /opt/mysql8/mysql-docker-compose.yml down -v

# 更新docker-compose
docker-compose -f /opt/mysql8/mysql-docker-compose.yml up -d --build
```

## mysql授权

```sh
# 进入mysql 容器内
mysql -uroot -p # 输入密码：root

use mysql; 
select host,user from user;

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY 'root123456';
FLUSH PRIVILEGES;

ALTER USER 'mySpider'@'%' IDENTIFIED WITH mysql_native_password BY '1q2w#E';
FLUSH PRIVILEGES;
```
