---
title: Ubuntu 22.04 LTS 中安装 Docker 和 Docker Compose
date: 2022-08-24 16:26:10
tags:
  - Docker Compose
  - Docker
  - Ubuntu
  - Linux
categories:
  - Docker Compose
---

## Docker 依赖项

为了安装并配置 Docker ，你的系统必须满足下列最低要求：

- 64 位 Linux 或 Windows 系统
- 如果使用 Linux ，内核版本必须不低于 3.10
- 能够使用 sudo 权限的用户
- 在你系统 BIOS 上启用了 VT（虚拟化技术）支持 on your system BIOS（参考: 如何查看 CPU 支持 虚拟化技术（VT））
- 你的系统应该联网

在 Linux ，在终端上运行以下命令验证内核以及架构详细信息：

```ps
uname -a
Linux pony 5.15.0-46-generic #49-Ubuntu SMP Thu Aug 4 18:03:25 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

<!--more-->

## 安装 Docker

### 更新 Ubuntu

打开终端，依次运行下列命令：

```sh
sudo apt update
sudo apt upgrade
sudo apt full-upgrade
```

### 添加 Docker 库

首先，安装必要的证书并允许 apt 包管理器使用以下命令通过 HTTPS 使用存储库：

```sh
sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release
```

然后，运行下列命令添加 Docker 的官方 GPG 密钥：

```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

添加 Docker 官方库：

```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

使用命令更新 Ubuntu 源列表：

```sh
sudo apt update
```

### 安装 Docker CE

安装最新 Docker CE:

```sh
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

当然你也可以安装其他版本 Docker 。运行下列命令检查可以安装的 Docker 版本：

```sh
apt-cache madison docker-ce
```

输出样例：

```sh
docker-ce | 5:20.10.17~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
docker-ce | 5:20.10.16~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
docker-ce | 5:20.10.15~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
docker-ce | 5:20.10.14~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
docker-ce | 5:20.10.13~3-0~ubuntu-jammy | https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
```

挑选上面列表中的任何版本进行安装。例如，安装 5:20.10.16~ 3-0 ~ubuntu-jammy 这个版本，运行：

```sh
sudo apt install docker-ce=5:20.10.16~3-0~ubuntu-jammy docker-ce-cli=5:20.10.16~3-0~ubuntu-jammy containerd.io
```

安装完成后，运行如下命令验证 Docker 服务是否在运行：

```sh
systemctl status docker
```

### Docker 常用命令

```sh
systemctl start docker #运行 Docker 服务
systemctl enable docker　#使 Docker 服务在每次重启时自动启动
docker version # 查看已安装的 Docker 版本
# 令会下载一个 Docker 测试镜像，
# 并在容器内执行一个 “hello_world” 样例程序
docker run hello-world
```

## 安装 Docker Compose

### 方式 1、使用二进制文件安装 Docker Compose

运行下列命令安装最新稳定的 Docker Compose 文件：

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/v2.6.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

如果有更新版本，只需要将上述命令中的 v2.6.1 替换为最新的版本号即可。请不要忘记数字前的 "v" 。

最后，使用下列命令赋予二进制文件可执行权限：

```sh
sudo chmod +x /usr/local/bin/docker-compose
```

运行下列命令检查安装的 Docker Compose 版本：

```sh
docker-compose version
Docker Compose version v2.6.1
```

### 方式 2、使用 Pip 安装 Docker Compose

安装 Pip

```sh
apt-get install python3-pip
```

安装 Pip 后，运行以下命令安装 Docker Compose。下列命令对于所有 Linux 发行版都是相同的！

```sh
pip install docker-compose
```

安装 Docker Compose 后，使用下列命令检查版本：

```sh
docker-compose --version
```

你将会看到类似下方的输出：

```sh
docker-compose version 2.6.1, build 8a1c60f6
```

参考：

[如何在 Ubuntu 22.04 LTS 中安装 Docker 和 Docker Compose](https://linux.cn/article-14871-1.html)

[How To Manage Python Packages Using PIP](https://ostechnix.com/manage-python-packages-using-pip/)
