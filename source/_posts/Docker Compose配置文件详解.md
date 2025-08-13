---
title: Docker Compose配置文件详解
date: 2018-05-11 14:38:31
tags:
- Docker
- Docker Compose
categories: 
- Docker Compose
---
# Docker Compose配置文件详解

参考官网：[https://docs.docker.com/compose/compose-file/#service-configuration-reference](https://docs.docker.com/compose/compose-file/#service-configuration-reference)

Compose文件是一个定义服务， 网络和 卷的YAML文件 。Compose文件的默认路径是。./docker-compose.yml

标准配置文件应该包含 version、services、networks 三大部分，其中最关键的就是 services 和 networks 两个部分。

提示：您可以对此文件使用a .yml或.yaml扩展名。他们都工作。

## image

``` docker
services:
  dotnetcoreapp1:
    image: dotnetcoreapp
```

在 services 标签下的第二级标签是 dotnetcoreapp1，这个名字是用户自己自定义，它就是服务名称。

image 则是指定服务的镜像名称或镜像 ID。如果镜像在本地不存在，Compose 将会尝试拉取这个镜像。

## build

在构建时应用的配置选项

build 可以指定为构建上下文的路径：

```docker
version: '3'
services:
  webapp:
    build: ./dir
```

``` docker
version: '3'
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
      args:
        buildno: 1
```

如果同时指定build和image，那么会在./dir中生成名为webapp和标记的镜像tag。

### context

可以是包含Dockerfile的目录的路径，也可以是到git存储库的URL。

当提供的值是相对路径时，它被解释为相对于撰写文件的位置。 这个目录也是发送到Docker守护进程的构建上下文。

使用生成的名称构建并标记它，然后使用该镜像。

``` docker
build:
  context: ./dir
```

### dockerfile

dockerfile作为备用，并且需要提供路径。

``` docker
build:
  context: .
  dockerfile: Dockerfile-alternate
```

### args

添加构建参数，这些参数是仅在构建过程中可访问的环境变量。

首先，指定Dockerfile：

``` docker
ARG buildno
ARG gitcommithash

RUN echo "Build number: $buildno"
RUN echo "Based on commit: $gitcommithash"
```

然后指定参数。 您可以传递一个映射或一个列表：

``` docker
build:
  context: .
  args:
    buildno: 1
    gitcommithash: cdc3b19

build:
  context: .
  args:
    - buildno=1
    - gitcommithash=cdc3b19
```

指定构建参数时可以省略该值，在这种情况下，构建时的值是构成运行环境中的值。

``` docker
args:
  - buildno
  - gitcommithash
```

注意：YAML 的布尔值（true, false, yes, no, on, off）必须要使用引号引起来（单引号、双引号均可），否则会当成字符串解析。

### cache_from

v3.2新添加

用于缓存解析的镜像列表

``` docker
build:
  context: .
  cache_from:
    - alpine:latest
    - corp/web_app:3.14
```

### labels

v3.3新添加

使用Docker标签将元数据添加到生成的图像。 您可以使用数组或字典。

建议您使用反向DNS标记来防止您的标签与其他软件使用的标签冲突。

``` docker
build:
  context: .
  labels:
    com.example.description: "Accounting webapp"
    com.example.department: "Finance"
    com.example.label-with-empty-value: ""

build:
  context: .
  labels:
    - "com.example.description=Accounting webapp"
    - "com.example.department=Finance"
    - "com.example.label-with-empty-value"
```

### shm_size

v3.5新添加

为此版本的容器设置/dev/shm分区的大小。 指定为表示字节数的整数值或表示字节值的字符串。

``` docker
build:
  context: .
  shm_size: '2gb'

build:
  context: .
  shm_size: 10000000
```

## cap_add, cap_drop

添加或删除容器功能

``` docker
cap_add:
  - ALL

cap_drop:
  - NET_ADMIN
  - SYS_ADMIN
```

## command

command 覆盖容器启动后默认执行的命令

``` docker
command: bundle exec thin -p 3000
```

该命令也可以是一个列表，方式类似于dockerfile：

``` docker
command: ["bundle", "exec", "thin", "-p", "3000"]
```

## configs

[configs命令](https://docs.docker.com/engine/swarm/configs/ "configs命令")

Grant access to configs on a per-service basis using the per-service configs configuration.

为每个服务配置配置为每个服务授予对配置的访问权限。支持两种不同的语法变体。

注意：配置必须已经存在或在此堆栈文件的顶级配置配置中定义，否则堆栈部署失败。

### SHORT SYNTAX

简短的语法变体只能指定配置​​名称

支持v3.3以上

以下示例使用简短语法将redis服务访问权限授予my_config和my_other_config配置。 my_config的值被设置为文件./my_config.txt的内容，my_other_config被定义为外部资源，这意味着它已经在Docker中定义，可以通过运行docker config create命令或通过另一个堆栈部署。如果外部配置不存在，堆栈部署将失败并显示配置未找到错误。

``` docker
version: "3.3"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - my_config
      - my_other_config
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

### LONG SYNTAX

长的语法提供了在服务的任务容器中如何创建配置的更细粒度。

source：Docker中配置的名称。

target：要在服务的任务容器中安装的文件的路径和名称。如果未指定，则默认为 `/<source>`。

uid和gid：在服务的任务容器中拥有安装的配置文件的数字UID或GID。如果未指定，则在Linux上均默认为0。 Windows不支持。

mode：安装在服务任务容器内的文件的权限，采用八进制表示法。例如，0444代表可读。缺省值为0444.由于Configs被挂载在临时文件系统中，因此它们不能写入，因此如果设置了可写位，它将被忽略。可执行位可以被设置。如果您不熟悉UNIX文件权限模式，则可能会发现此权限计算器很有用。

以下示例将容器中my_config的名称设置为redis_config，将模式设置为0440（可读组），并将用户和组设置为103. redis服务无法访问my_other_config配置

``` docker
version: "3.3"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    configs:
      - source: my_config
        target: /redis_config
        uid: '103'
        gid: '103'
        mode: 0440
configs:
  my_config:
    file: ./my_config.txt
  my_other_config:
    external: true
```

您可以授予多个配置的服务访问权限，您可以混合使用长短语法。定义配置并不意味着授予服务访问权限。

## container_name

指定自定义容器名称

由于Docker容器名称必须是唯一的，因此如果您指定了自定义名称，则无法将服务扩展到1个容器之外。 试图这样做会导致错误。

``` docker
container_name: my-web-container
```

注意：当使用（版本3）Compose文件在群集模式下部署堆栈时，忽略此选项。

## deploy

指定与部署和运行服务相关的配置。 这只在部署到使用docker stack部署的群集时才生效，并且被docker-compose up和docker-compose run忽略。

```docker
version: '3'
services:
  redis:
    image: redis:alpine
    deploy:
      replicas: 6
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
```

### ENDPOINT_MODE

指定连接到群组的外部客户端的服务发现方法。

v3.3独有

`endpoint_mode：vip` - Docker为服务分配一个虚拟IP（VIP），作为客户端到达网络服务的“前端”。 Docker在客户端和可用的工作节点之间为服务路由请求，而客户端不知道有多少节点参与服务或其IP地址或端口。 （这是默认设置。）

`endpoint_mode：dnsrr` - DNS轮询（DNSRR）服务发现不使用单个虚拟IP。 Docker为服务设置DNS条目，使得服务名称的DNS查询返回一个IP地址列表，并且客户端直接连接到其中的一个。 如果您想使用自己的负载平衡器，或者混合Windows和Linux应用程序，则DNS轮询功能非常有用。

```docker
version: "3.3"

services:
  wordpress:
    image: wordpress
    ports:
      - "8080:80"
    networks:
      - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: vip

  mysql:
    image: mysql
    volumes:
       - db-data:/var/lib/mysql/data
    networks:
       - overlay
    deploy:
      mode: replicated
      replicas: 2
      endpoint_mode: dnsrr

volumes:
  db-data:

networks:
  overlay:
```

[docker service create](https://docs.docker.com/engine/reference/commandline/service_create/)

### LABELS

指定服务的标签。 这些标签仅在服务上设置，而不在服务的任何容器上设置。

```docker
version: "3"
services:
  web:
    image: web
    deploy:
      labels:
        com.example.description: "This label will appear on the web service"
```

要在容器上设置标签，请在部署之外使用标签键：

```docker
version: "3"
services:
  web:
    image: web
    labels:
      com.example.description: "This label will appear on all containers for the web service"
```

### MODE

全局（每个群集节点只有一个容器）或复制（指定数量的容器）。 默认值被复制。 （要了解更多信息，请参阅swarm主题中的复制和全局服务。）

```docker
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    deploy:
      mode: global
```

### PLACEMENT

指定约束和偏好的位置。

```docker
version: '3.3'
services:
  db:
    image: postgres
    deploy:
      placement:
        constraints:
          - node.role == manager
          - engine.labels.operatingsystem == ubuntu 14.04
        preferences:
          - spread: node.labels.zone
```

### REPLICAS

如果服务被复制（这是默认设置），请指定在任何给定时间应该运行的容器数量。

```docker
version: '3'
services:
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 6
```

### RESOURCES

配置资源限制。

注意：这会替换版本3之前的Compose文件（cpu_shares，cpu_quota，cpuset，mem_limit，memswap_limit，mem_swappiness）中的非群集模式的旧资源约束选项，如升级2.x版至3.x中所述。

在这个一般的例子中，Redis服务限制使用不超过50M的内存和0.50（50％）的可用处理时间（CPU），并且具有20M的内存和0.25个CPU时间保留（总是可用）。

```docker
version: '3'
services:
  redis:
    image: redis:alpine
    deploy:
      resources:
        limits:
          cpus: '0.50'
          memory: 50M
        reservations:
          cpus: '0.25'
          memory: 20M
```

这里描述的选项特定于部署密钥和群集模式。 如果要为非群部署设置资源约束，请使用Compose文件格式版本2 CPU，内存和其他资源选项。

** 内存异常（OOME）**
如果您的服务或容器尝试使用比系统可用的内存更多的内存，则可能会遇到内存异常（OOME），并且容器或Docker守护程序可能会被内核OOM杀手所杀。 要防止发生这种情况，请确保您的应用程序在具有足够内存的主机上运行，并且请参阅了解耗尽内存的风险。

### RESTART_POLICY

配置是否以及如何在退出时重新启动容器。 取代重新启动。

`condition`:无，失败或任何（默认：任何）。
`delay`:在重启尝试之间等待多长时间，指定为持续时间（默认值：0）。
`max_attempts`:放弃之前尝试重新启动容器的次数（默认值：永不放弃）。 如果重新启动在配置的窗口内没有成功，则此尝试不计入配置的max_attempts值。 例如，如果max_attempts设置为'2'，并且第一次尝试重新启动失败，则可能会尝试重新启动两次以上。
`window`：在决定重新启动是否成功之前等待多久，指定为持续时间（默认值：立即决定）。

```docker
version: "3"
services:
  redis:
    image: redis:alpine
    deploy:
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
        window: 120s
```

### UPDATE_CONFIG

配置如何更新服务。 用于配置滚动更新。

`parallelism`：一次更新容器的数量。
`delay`：更新一组容器之间的等待时间。
`failure_action`：更新失败时该怎么做。 继续，回滚或暂停之一（默认：暂停）。
`monitor`：每次任务更新后监视失败的时间（ns | us | ms | s | m | h）（默认为0）。
`max_failure_ratio`：在更新期间容忍的失败率。
`order`：更新期间的操作顺序。 停止优先（旧任务在开始新任务之前停止）或者先启动（首先启动新任务，并且正在运行的任务短暂重叠）（默认停止优先）注意：只支持v3.4及更高版本。

order支持v3.4或更高版本

```docker
version: '3.4'
services:
  vote:
    image: dockersamples/examplevotingapp_vote:before
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
        order: stop-first
```





## devices

设备映射列表。 使用与--device docker客户端创建选项相同的格式。

``` docker
devices:
  - "/dev/ttyUSB0:/dev/ttyUSB0"
```

注意：当使用（版本3）Compose文件在群集模式下部署堆栈时，忽略此选项。

## depends_on

服务之间的快速依赖关系，服务依赖关系导致以下行为：

`docker-compose up` 以依赖顺序启动服务。 在以下示例中，db和redis在web之前启动。

`docker-up up SERVICE` 自动包含SERVICE的依赖关系。 在下面的例子中，docker-compose up web也会创建并启动db和redis。

```docker
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

使用depends_on时有几件事要注意：

在启动web之前，depends_on不会等待db和redis“准备就绪” - 只有在它们启动之前。 如果您需要等待服务准备就绪，请参阅控制启动顺序以了解有关此问题的更多信息以及解决此问题的策略。

版本3不再支持depends_on的条件形式。

使用版本3撰写文件在群集模式下部署堆栈时，将忽略depends_on选项。

## dns

自定义DNS服务器。 可以是单个值或列表。

```docker
dns: 8.8.8.8
dns:
  - 8.8.8.8
  - 9.9.9.9
```

## dns_search

自定义DNS搜索域。 可以是单个值或列表。

```docker
dns_search: example.com
dns_search:
  - dc1.example.com
  - dc2.example.com
```

## tmpfs

v2.0以上

在容器中装入一个临时文件系统。 可以是单个值或列表。

```docker
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```

注意：在使用（版本3-3.5）撰写文件的群集模式下部署堆栈时，忽略此选项。

v3.6以上

在容器中装入一个临时文件系统。 Size参数以字节为单位指定tmpfs安装的大小。 无限制默认。

``` docker
- type: tmpfs
     target: /app
     tmpfs:
       size: 1000
```

## entrypoint

``` docker
entrypoint: /code/entrypoint.sh
```

```docker
entrypoint:
    - php
    - -d
    - zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20100525/xdebug.so
    - -d
    - memory_limit=-1
    - vendor/bin/phpunit
```

注意：设置入口点都会使用ENTRYPOINT Dockerfile指令覆盖服务映像上设置的任何默认入口点，并清除映像上的任何默认命令 - 这意味着如果Dockerfile中存在CMD指令，则它将被忽略。

## env_file

从文件添加环境变量。 可以是单个值或列表。

如果您已经使用`docker-compose -f FILE`指定了Compose文件，则env_file中的路径与该文件所在的目录相关。

环境部分中声明的环境变量会覆盖这些值 - 即使这些值为空或未定义，也是如此。

```docker
env_file: .env

env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```

撰写Compose env文件中的每行都是VAR = VAL格式。 以＃开始的行被视为注释并被忽略。 空行也被忽略。

```docker
# Set Rails/Rack environment
RACK_ENV=development
```

注意：如果您的服务指定了构建选项，则在构建期间，环境文件中定义的变量不会自动显示。 使用build的args子选项来定义构建时环境变量。

定义的值并不会始终保持不变，列表中的值按照顺序执行，以最后值为准。

## environment

添加环境变量。 您可以使用数组或字典。

 任何布尔值; true, false, yes no，需要用引号括起来以确保它们不被YML解析器转换为True或False。

 ```docker
 environment:
  RACK_ENV: development
  SHOW: 'true'
  SESSION_SECRET:

environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```

注意：如果您的服务指定了构建选项，则在构建过程中定义的变量不会自动显示。 使用build的args子选项来定义构建时环境变量。

## expose

公开端口而不将它们发布到主机 - 它们只能被链接服务访问。 只能指定内部端口。

```docker
expose:
 - "3000"
 - "8000"
```

## external_links

链接到此Docker-compose.yml之外甚至Compose之外的容器，尤其是提供共享或公共服务的容器。 在指定容器名称和链接别名（CONTAINER：ALIAS）时，external_links遵循类似于旧版选项链接的语义。

```docker
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```

## extra_hosts

添加主机名映射。 使用与docker客户端--add-host参数相同的值。

```docker
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

具有ip地址和主机名的条目在容器内的`/etc/hosts`中为该服务创建，例如：

```docker
162.242.195.82  somehost
50.31.209.229   otherhost
```

## healthcheck

v2.1以上

配置运行的检查以确定此服务的容器是否“健康”。 查看[HEALTHCHECK Dockerfile](https://docs.docker.com/engine/reference/builder/#usage "HEALTHCHECK Dockerfile")指令的文档以获取有关健康检查如何工作的详细信息。

```docker
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

``` docker
# Hit the local web app
test: ["CMD", "curl", "-f", "http://localhost"]

# As above, but wrapped in /bin/sh. Both forms below are equivalent.
test: ["CMD-SHELL", "curl -f http://localhost || exit 1"]
test: curl -f https://localhost || exit 1
```

要禁用图像设置的任何默认健康检查，您可以使用disable：true。 这相当于指定测试：[“NONE”]。

```docker
healthcheck:
  disable: true
```

## Specifying durations

一些配置选项（如检查的间隔和超时子选项）以如下格式的字符串形式接受持续时间：

```docker
2.5s
10s
1m30s
2h32m
5h34m56s
```

支持的单位是us，ms，s，m和h。

## isolation

指定容器的隔离技术。 在Linux上，唯一支持的值是默认值。 在Windows上，可接受的值是默认值，进程和hyperv。 有关详细信息，请参阅Docker Engine文档。

## labels

使用Docker标签将元数据添加到容器。 您可以使用数组或字典。

建议您使用反向DNS标记来防止您的标签与其他软件使用的标签冲突。

``` docker
labels:
  com.example.description: "Accounting webapp"
  com.example.department: "Finance"
  com.example.label-with-empty-value: ""

labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```

## logging

记录服务的配置。

```docker
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```

驱动程序名称指定服务容器的日志记录驱动程序，与docker run的--log-driver选项（此处记录）一样。

默认值是json-file。

```docker
driver: "json-file"
driver: "syslog"
driver: "none"
```

注意：只有`json-file`和`journald`驱动程序才能使日志直接从`docker-compose up`和`docker-compose logs`。 使用任何其他驱动程序不会打印任何日志。

使用选项键指定日志记录驱动程序的日志记录选项，就像`docker run`的--log-opt选项一样。

```docker
driver: "syslog"
options:
  syslog-address: "tcp://192.168.0.42:123"
```

默认驱动程序json-file具有限制存储日志量的选项。 为此，请使用键值对来获得最大存储大小和最大文件数量：

```docker
options:
  max-size: "200k"
  max-file: "10"
```

上面显示的示例将存储日志文件，直到它们达到200kB的最大大小，然后旋转它们。 存储的单个日志文件的数量由最大文件值指定。 随着日志增长超出最大限制，旧日志文件将被删除以允许存储新日志。

以下是限制日志存储的示例docker-compose.yml文件：

```docker
services:
  some-service:
    image: some-service
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

## network_mode

网络模式。 使用与docker客户端相同的值--net参数，以及特殊的表单服务：[服务名称]。

``` docker
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

network_mode：“host”不能与链接混合使用。

## networks

加入网络，引用顶级网络密钥下的条目。

```docker
services:
  some-service:
    networks:
     - some-network
     - other-network
```

### ALIASES

网络上此服务的别名（备用主机名）。 同一网络上的其他容器可以使用服务名称或别名来连接到某个服务的容器。

由于别名是网络范围的，相同的服务可以在不同的网络上具有不同的别名。

注意：网络范围的别名可以由多个容器共享，甚至可以由多个服务共享。 如果是，那么名称解析的确切容器的名称就不能保证。

```docker
services:
  some-service:
    networks:
      some-network:
        aliases:
         - alias1
         - alias3
      other-network:
        aliases:
         - alias2
```

```docker
version: '2'

services:
  web:
    build: ./web
    networks:
      - new

  worker:
    build: ./worker
    networks:
      - legacy

  db:
    image: mysql
    networks:
      new:
        aliases:
          - database
      legacy:
        aliases:
          - mysql

networks:
  new:
  legacy:
```

### IPV4_ADDRESS, IPV6_ADDRESS

加入网络时，为此服务的容器指定一个静态IP地址。

顶级网络部分中的相应网络配置必须具有覆盖每个静态地址的具有子网配置的ipam块。 如果需要IPv6寻址，则必须设置enable_ipv6选项，并且必须使用版本2.x Compose文件，如下所示。

注意：这些选项目前不在群集模式下工作。

```docker
version: '2.1'

services:
  app:
    image: busybox
    command: ifconfig
    networks:
      app_net:
        ipv4_address: 172.16.238.10
        ipv6_address: 2001:3984:3989::10

networks:
  app_net:
    driver: bridge
    enable_ipv6: true
    ipam:
      driver: default
      config:
      -
        subnet: 172.16.238.0/24
      -
        subnet: 2001:3984:3989::/64
```

## pid

```docker
pid: "host"
```

将PID模式设置为主机PID模式。 这将打开容器与主机操作系统之间的共享PID地址空间。 使用此标志启动的容器可以访问和操作裸机的名称空间中的其他容器，反之亦然。

## ports

公开端口

注意：端口映射与network_mode：host不兼容

### SHORT SYNTAX

既可以指定两个端口（HOST：CONTAINER），也可以指定容器端口（选择临时主机端口）。

注意：以HOST：CONTAINER格式映射端口时，使用低于60的容器端口时可能会遇到错误结果，因为YAML会将格式为xx：yy的数字解析为基数为60的值。 出于这个原因，我们建议始终明确指定您的端口映射为字符串。

```docker
ports:
 - "3000"
 - "3000-3005"
 - "8000:8000"
 - "9090-9091:8080-8081"
 - "49100:22"
 - "127.0.0.1:8001:8001"
 - "127.0.0.1:5000-5010:5000-5010"
 - "6060:6060/udp"
```

### LONG SYNTAX

长格式语法允许配置不能以简短形式表示的附加字段。

target：容器内的端口
published：公开曝光的港口
protocol：端口协议（tcp或udp）
mode：用于在每个节点上发布主机端口的主机，或用于群集模式端口的入口以进行负载平衡。

```docker
ports:
  - target: 80
    published: 8080
    protocol: tcp
    mode: host
```

注意：长语法在v3.2中是新的

## secrets

``` docker
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - my_secret
      - my_other_secret
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

```docker
version: "3.1"
services:
  redis:
    image: redis:latest
    deploy:
      replicas: 1
    secrets:
      - source: my_secret
        target: redis_secret
        uid: '103'
        gid: '103'
        mode: 0440
secrets:
  my_secret:
    file: ./my_secret.txt
  my_other_secret:
    external: true
```

## security_opt

覆盖每个容器的默认标签方案。

``` docker
security_opt:
  - label:user:USER
  - label:role:ROLE
```

## stop_grace_period

指定在发送SIGKILL之前，如果试图停止一个容器（如果它没有处理SIGTERM）（或者使用stop_signal指定了任何停止信号），请等待多久。 指定为持续时间。

```docker
stop_grace_period: 1s
stop_grace_period: 1m30s
```

默认情况下，停止在发送SIGKILL之前等待10秒钟容器退出。

## stop_signal

设置一个替代信号来停止容器。 默认停止使用SIGTERM。 使用stop_signal设置替代信号会导致停止发送该信号。

```docker
stop_signal: SIGUSR1
```

## sysctls

在容器中设置的内核参数。 您可以使用数组或字典。

```docker
sysctls:
  net.core.somaxconn: 1024
  net.ipv4.tcp_syncookies: 0

sysctls:
  - net.core.somaxconn=1024
  - net.ipv4.tcp_syncookies=0
```

## ulimits

覆盖容器的默认限制。 您可以将单个限制指定为整数，也可以将软限制/硬限制指定为映射。

```docker
ulimits:
  nproc: 65535
  nofile:
    soft: 20000
    hard: 40000
```

## userns_mode

```docker
userns_mode: "host"
```

如果Docker守护程序配置了用户命名空间，则禁用此服务的用户命名空间。

## volumes

装载主机路径或命名卷，将其指定为服务的子选项。

您可以将主机路径作为单个服务的定义的一部分进行安装，并且无需在顶级卷密钥中定义它。

但是，如果要跨多个服务重用卷，请在顶级卷密钥中定义一个命名卷。 将命名卷与服务，群集和堆栈文件一起使用。

注意：顶级卷密钥定义了一个命名卷，并从每个服务的卷列表中引用它。 这会替换早期版本的Compose文件格式中的volumes_from。 有关卷的一般信息，请参阅使用卷和卷插件。

此示例显示Web服务正在使用的命名卷（mydata），以及为单个服务（数据库服务卷下的第一个路径）定义的绑定挂载。 数据库服务还使用名为dbdata的命名卷（数据库服务卷下的第二个路径），但使用旧的字符串格式定义它以装载命名卷。 如图所示，命名卷必须列在顶级卷密钥下。

```docker
version: "3.2"
services:
  web:
    image: nginx:alpine
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

  db:
    image: postgres:latest
    volumes:
      - "/var/run/postgres/postgres.sock:/var/run/postgres/postgres.sock"
      - "dbdata:/var/lib/postgresql/data"

volumes:
  mydata:
  dbdata:
```

### SHORT SYNTAX

可以选择在主机上（HOST：CONTAINER）或访问模式（HOST：CONTAINER：ro）指定路径。

您可以在主机上挂载相对路径，该路径相对于正在使用的Compose配置文件的目录进行扩展。 相对路径应始终以

`.`或`..`。

```docker
volumes:
  # Just specify a path and let the Engine create a volume
  - /var/lib/mysql

  # Specify an absolute path mapping
  - /opt/data:/var/lib/mysql

  # Path on the host, relative to the Compose file
  - ./cache:/tmp/cache

  # User-relative path
  - ~/configs:/etc/configs/:ro

  # Named volume
  - datavolume:/var/lib/mysql
```

### LONG SYNTAX

`type`：装载类型卷，绑定或tmpfs
`source`：装入源，主机上用于绑定装入的路径或顶级卷密钥中定义的卷的名称。 不适用于tmpfs安装。
`target`：卷所在的容器中的路径
`read_only`：标志将卷设置为只读
`bind`：配置额外的绑定选项
  `propagation`：用于绑定的传播模式
`volume`：配置额外的volume选项
  `nocopy`：标志，用于在卷创建时禁止从容器复制数据
`tmpfs`：配置附加的tmpfs选项
  `size`：tmpfs的大小，以字节为单位

```docker
version: "3.2"
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - type: volume
        source: mydata
        target: /data
        volume:
          nocopy: true
      - type: bind
        source: ./static
        target: /opt/app/static

networks:
  webnet:

volumes:
  mydata:
```

Note: The long syntax is new in v3.2

** 服务，群集和堆栈文件的卷**

在使用服务，群集和docker-stack.yml文件时，请记住支持服务的任务（容器）可以部署在群集中的任何节点上，并且每次更新服务时都可能是不同的节点。

在缺少指定源的命名卷的情况下，Docker为支持服务的每个任务创建一个匿名卷。关联的容器被移除后，匿名卷不会保留。

如果您希望数据持久存在，请使用可识别多主机的命名卷和卷驱动程序，以便可以从任何节点访问数据。或者，对该服务设置约束，以便将其任务部署在具有该卷的节点上。

作为一个例子，Docker Labs中votingapp示例的docker-stack.yml文件 定义了一个称为运行数据库的服务。它被配置为一个命名卷来保存群体上的数据， 并且仅限于在节点上运行。这是来自该文件的相关剪辑：db postgres manager

```docker
version: "3"
services:
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
```

## restart

no是默认的重启策略，在任何情况下都不会重启容器。 总是指定时，容器总是重新启动。 如果退出代码指示出现故障错误，则故障中策略会重新启动容器。

```docker
restart: "no"
restart: always
restart: on-failure
restart: unless-stopped
```

## domainname, hostname, ipc, mac_address, privileged, read_only, shm_size, stdin_open, tty, user, working_dir

```docker
user: postgresql
working_dir: /code

domainname: foo.com
hostname: foo
ipc: host
mac_address: 02:42:ac:11:65:43

privileged: true


read_only: true
shm_size: 64M
stdin_open: true
tty: true
```

## Specifying durations

一些配置选项（如interval和timeout ）以如下格式的字符串形式接受持续时间：

```docker
2.5s
10s
1m30s
2h32m
5h34m56s
```

## Volume configuration reference

虽然可以将文件中的卷声明为服务声明的一部分，但本节允许您创建可在多个服务中重复使用的命名卷（不依赖于volumes_from），并且可以使用docker命令行轻松检索和检查 或API。 有关更多信息，请参阅docker卷子命令文档。

有关卷的一般信息，请参阅使用卷和卷插件。

以下是一个双服务设置的示例，其中数据库的数据目录与另一个服务共享为一个卷，以便可以定期进行备份：

```docker
version: "3"

services:
  db:
    image: db
    volumes:
      - data-volume:/var/lib/db
  backup:
    image: backup-service
    volumes:
      - data-volume:/var/lib/backup/data

volumes:
  data-volume:
```

顶级卷密钥下的条目可以为空，在这种情况下，它使用由引擎配置的默认驱动程序（在大多数情况下，这是本地驱动程序）。 或者，您可以使用以下键配置它：

### driver

指定应该为此卷使用哪个卷驱动程序。 默认为Docker引擎已经配置使用的驱动程序，在大多数情况下它是本地的。 如果驱动程序不可用，则当docker-compose up尝试创建卷时，引擎会返回错误。

```docker
 driver: foobar
```

### driver_opts

指定一个选项列表作为传递给该卷的驱动程序的键值对。 这些选项是依赖于驱动程序的 - 请查阅驱动程序的文档以获取更多信息。 可选的。

```docker
 driver_opts:
   foo: "bar"
   baz: 1
```

### external

如果设置为true，则指定该卷已在Compose之外创建。 docker-compose up不会尝试创建它，并在它不存在时引发错误。

外部不能与其他卷配置键（驱动程序，driver_opts）一起使用。

在下面的示例中，Compose不是试图创建一个名为[projectname] _data的卷，而是查找一个简单称为data的现有卷，并将其挂载到数据库服务的容器中。

```docker
version: '2'

services:
  db:
    image: postgres
    volumes:
      - data:/var/lib/postgresql/data

volumes:
  data:
    external: true
```

external.name在版本3.4文件格式中被弃用，而使用name。

您还可以在Compose文件中单独指定用于引用它的名称来指定卷的名称：

```docker
volumes:
  data:
    external:
      name: actual-name-of-volume
```

** 外部卷始终使用Docker堆栈部署来创建**

如果您使用docker stack deploy以swarm模式启动应用程序（而不是docker组合），则创建不存在的外部卷。 在群集模式下，当服务定义时会自动创建一个卷。 由于服务任务在新节点上进行安排，因此swarmkit会在本地节点上创建卷

### labels

使用Docker标签将元数据添加到容器。 您可以使用数组或字典。

建议您使用反向DNS标记来防止您的标签与其他软件使用的标签冲突。

```docker
labels:
  com.example.description: "Database volume"
  com.example.department: "IT/Ops"
  com.example.label-with-empty-value: ""

labels:
  - "com.example.description=Database volume"
  - "com.example.department=IT/Ops"
  - "com.example.label-with-empty-value"
```

### name

为此卷设置一个自定义名称。 名称字段可用于引用包含特殊字符的网络。 该名称按原样使用，不会与堆栈名称一起作用。

```docker
version: '3.4'
volumes:
  data:
    name: my-app-data
```

```docker
version: '3.4'
volumes:
  data:
    external: true
    name: my-app-data
```