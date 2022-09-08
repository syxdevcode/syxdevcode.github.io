---
title: Ubuntu22-04启用UFW防火墙
date: 2022-09-08 13:23:19
tags:
  - Ubuntu
  - Linux基础
  - Linux
  - iptables
  - ufw
  - 安全策略
categories:
  - 防火墙
---

UFW，或称 Uncomplicated Firewall，是 iptables 的一个接口，为不熟悉防火墙概念的初学者提供了易于使用的界面，同时支持 IPv4 和 IPv6，广受欢迎。

## 更新 Ubuntu

```shell
sudo apt update && sudo apt upgrade -y
```

注意：docker 自动在防火墙列表中添加了开放端口的规则. 所以根本就没走到 ufw 端口就被放行了。

## 启用、安装或删除 UFW

默认情况下，应安装 UFW，但如果已将其删除，请重新安装 UFW。

```shell
sudo apt install ufw -y
```

## 检查防火墙状态

```shell
# 服务状态
systemctl status ufw

# UFW 防火墙的状态
ufw status
```

## 设置 SSH 防火墙规则

启用防火墙，它将阻止所有传入连接并允许所有传出连接。 这将立即帮助保护您的系统。

对于服务器用户或任何其他使用 SSH 远程连接会话的用户，您可以将自己锁定。 幸运的是，您可以在服务未激活时添加 UFW 规则，从而允许 SSH 服务。

```shell
sudo ufw allow ssh
```

<!--more-->

## 启用/禁用防火墙

### 启用防火墙

```shell
sudo ufw enable
```

输出示例：

Firewall is active and enabled on system startup

默认情况下，所有传入流量都会被自动阻止，一旦防火墙启用，所有出站流量都将被允许。 这将立即通过阻止任何人远程连接到您的系统来保护您的系统。

接下来，通过重新使用重新检查您的 Ubuntu 防火墙 ufw 状态命令。

```shell
sudo ufw status

#　运行附加详细信息的 ufw status 命令以获得更详细的视图。
sudo ufw status verbose
```

### 禁用防火墙

```shell
# 禁用 UFW 防火墙
sudo ufw disable
```

### 完全删除 UFW（不推荐）

```shell
sudo apt remove ufw --purge
```

除非您有可靠的选择或知道如何使用 IPTables，否则不要删除 UFW，尤其是在运行连接到公共的服务器环境时。 这将是灾难性的。

## 检查 UFW 状态

```shell
sudo ufw status numbered
```

输出：

```shell
root@test:/lims# ufw status numbered
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

## 设置/配置 UFW 默认策略

UFW 防火墙的默认策略是拒绝所有传入连接，只允许出站连接到系统。 通常最安全的默认方式是没有人可以访问您的服务器，除非您允许 IP 地址/范围、程序、端口或所有这些的组合。 默认情况下，您的系统可以访问外部，除非您有特定的安全要求，否则不应对其进行调整。

可以在 `/etc/default/ufw` 找到默认的 UFW 防火墙策略。

**拒绝所有传入连接：**

```shell
sudo ufw default deny incoming
```

**允许所有传出连接：**

```shell
sudo ufw default allow outgoing
```

这已在启用时设置为默认规则，但您可以使用相同的原则来更改它们以满足您的目的。

**阻止所有传出连接：**

例如，默认情况下阻止所有传入通信，但您希望阻止所有传出并仅允许出站连接，然后使用以下命令。

```shell
sudo ufw default deny outgoing
```

这是一个极端的措施； 对于普通服务器和台式机来说，阻止传入连接通常就足够了，但特定环境可以从额外的安全预防措施中受益。 缺点是您需要维护所有传出连接，这可能很耗时，而且需要不断设置新规则。

## 查看 UFW 应用程序配置文件

要显示所有应用程序配置文件，您可以键入以下内容。

```shell
sudo ufw app list
```

应用程序配置文件的一个方便功能是查找有关 UFW 应用程序列表中列出的服务的更多信息。

应用程序的一般描述和它使用的端口的打印输出。 当您调查开放端口并且不确定它们与哪些应用程序相关以及它们的作用时，这是一个方便的功能。

为此，请键入以下命令以查找有关现有配置文件的更多信息。

```shell
sudo ufw app info 'Nginx Full'
```

输出：

```shell
root@test:/lims# sudo ufw app info 'Nginx Full'
Profile: Nginx Full
Title: Web Server (Nginx, HTTP + HTTPS)
Description: Small, but very powerful and efficient web server

Ports:
  80,443/tcp
```

## 在 UFW 上允许/启用 IPv6

如果您的系统配置了 IPv6，则需要确保 UFW 配置了 IPv6 和 IPv4 支持。 默认情况下，这应该是自动启用的； 但是，您应该检查并在需要时对其进行修改。 您可以在下面执行此操作。

打开默认的 UFW 防火墙文件。

```shell
sudo nano /etc/default/ufw
```

如果未设置，请将以下行调整为 yes。

```shell
IPV6=yes
```

CTRL + O 保存对文件的新更改，然后按 CTRL + X 退出文件。

现在重新启动 UFW 防火墙服务以使更改生效。

```shell
sudo systemctl restart ufw
```

## 允许/启用 UFW 端口

使用 UFW，您可以打开防火墙中的特定端口以允许为特定应用程序指定的连接。 您可以为应用程序设置自定义规则。 这个规则的一个很好的例子是设置一个监听端口的网络服务器 80 (HTTP) and 443 (HTTPS) 默认情况下。

### 允许 HTTP 端口 80

**通过应用程序配置文件允许：**

```shell
sudo ufw allow 'Nginx HTTP'
```

**按服务名称允许：**

```shell
sudo ufw allow http
```

**按端口号允许：**

```shell
sudo ufw allow 80/tcp
```

### 允许 HTTPS 端口 443

**通过应用程序配置文件允许：**

```shell
sudo ufw allow 'Nginx HTTPS'
```

**按服务名称允许：**

```shell
sudo ufw allow https
```

**按端口号允许：**

```shell
sudo ufw allow 443/tcp
```

请注意，您可以使用以下命令默认启用所有规则。

```shell
sudo ufw allow 'Nginx Full'
```

### UFW 允许端口范围

UFW 可以允许访问端口范围。 打开端口范围时，您必须确定端口协议。

**允许使用 TCP 和 UDP 的端口范围：**

```shell
sudo ufw allow 6500:6800/tcp
sudo ufw allow 6500:6800/udp
```

或者，您可以一次允许多个端口，但范围可能更容易访问。

```shell
sudo ufw allow 6500, 6501, 6505, 6509/tcp
sudo ufw allow 6500, 6501, 6505, 6509/udp
```

## 允许/启用 UFW 上的远程连接

### UFW 允许特定 IP 地址

例如，要允许指定 IP 地址，您在内部网络上并要求系统一起通信，请使用以下命令。

```shell
sudo ufw allow from 192.168.55.131
```

## UFW 允许特定端口上的特定 IP 地址

启用 IP 以通过定义的端口连接到您的系统 （示例端口 “3900”），键入以下内容。

```shell
sudo ufw allow from 192.168.55.131 to any port 3900
```

### 允许子网连接到指定端口

如果您需要从 IP 范围子网到特定端口的整个连接范围，您可以通过创建以下规则来启用此功能。

```shell
sudo ufw allow from 192.168.1.0/24 to any port 3900
```

这将允许从 192.168.1.1 到 192.168.1.254 的所有 IP 地址连接到端口 3900。

### 允许特定网络接口

例如，允许连接到特定的网络接口， “Eth2” 到指定的端口 3900。您可以通过创建以下规则来实现此目的。

```shell
sudo ufw allow in on eth2 to any port 3900
```

## 拒绝/阻止 UFW 上的远程连接

根据 UFW 的默认设置策略，安装后，所有传入连接都设置为 “否定。” 这将拒绝所有传入流量，除非您创建规则以允许连接通过。

但是，您已经注意到一个特定的 IP 地址在您的日志中不断攻击您。 用以下方法阻止它。

```shell
sudo ufw deny from 203.13.56.121
```

黑客使用来自同一子网的多个 IP 地址来入侵您。 创建以下内容以阻止。

```shell
sudo ufw deny from 203.13.56.121/24
```

您可以创建特定规则来拒绝对特定端口的访问。 键入以下示例。

```shell
sudo ufw deny from 203.13.56.121/24 to any port 80
sudo ufw deny from 203.13.56.121/24 to any port 443
```

## 删除/删除 UFW 规则

要使用规则编号删除 UFW 规则，您必须通过键入以下内容列出规则编号。

```shell
sudo ufw status numbered
```

该示例将删除第三条规则 IP 地址 1.1.1.1， 上面突出显示。

在您的终端中键入以下内容。

```shell
sudo ufw delete 3
```

## 访问和查看 UFW 日志

默认情况下，UFW 日志记录设置为低，这对于大多数桌面系统来说都很好。 但是，服务器可能需要更高级别的日志记录。

要将 UFW 日志记录设置为低（默认）：

```shell
sudo ufw logging low
```

要将 UFW 日志记录设置为中等监控：

```shell
sudo ufw logging medium
```

要将 UFW 日志记录设置为高：

```shell
sudo ufw logging high
```

最后一个选项是完全禁用日志记录，确保您对此感到满意并且不需要日志检查。

```shell
sudo ufw logging off
```

要查看 UFW 日志，它们保存在默认位置 `/var/log/ufw.log` 。

查看实时日志的一种简单快捷的方法是使用 tail 命令。

```shell
sudo ufw tail -f /var/log/ufw.log
```

或者，您可以使用 -n.

```shell
sudo ufw tail /var/log/ufw.log -n 30
```

这将打印出日志的最后 30 行。 您可以使用 GREP 和其他排序命令进一步微调。

## 测试 UFW 规则

高度关键的系统，在玩弄防火墙设置时是一个不错的选择，可以添加 – 试运行标志. 这允许查看可能发生但未处理的更改的示例。

```shell
sudo ufw --dry-run enable
```

禁用 – 试运行标志, 使用以下命令。

## 重置 UFW 规则

要将您的防火墙重置回其原始状态并将所有传入阻止和传出设置为允许，请键入以下内容进行重置。

```shell
sudo ufw reset
```

确认重置，输入以下内容：

```shell
sudo ufw status
```

输出应该是：

```shell
Status: inactive
```

UFW 防火墙重置后，您现在需要重新启用防火墙并开始添加规则的整个过程。 如果可能，应谨慎使用重置命令。

## 查找/搜索所有开放端口（安全检查）

大多数系统没有意识到它们可以打开端口。 在互联网上每个 IP 地址每天都被扫描的时代，观察幕后发生的事情至关重要。

最好的选择是安装 Nmap，然后使用这个著名的应用程序列出打开的端口。

```shell
sudo apt install nmap -y
```

接下来，找到系统的内部 IP 地址。

```shell
hostname -I
```

输出示例：

```shell
192.168.50.214
```

现在使用以下 Nmap 命令和服务器的 IP 地址。

```shell
nmap 192.168.50.214
```

如上，除 80 端口外，所有端口都关闭，这是 UFW 规则中允许的，所以这是令人满意的。

但是，如果您在关闭或阻止端口之前发现端口已打开，如果您不确定，请先调查它们是什么，因为这可能会中断服务，或者更糟糕的情况是，将您锁定在服务器之外。

## 安装 UFW 防火墙 GUI(可选)

```shell
sudo apt install gufw -y
```

接下来，转到左上角并按照 活动 > 显示应用程序 > 防火墙配置 调出 GUI(firewall)。

参考：

[如何在 Ubuntu 22.04 LTS 上安装和配置 UFW 防火墙](https://zh-cn.linuxcapable.com/%E5%9C%A8-ubuntu-22-04-%E4%B8%8A%E5%AE%89%E8%A3%85%E5%90%AF%E7%94%A8%E9%85%8D%E7%BD%AE-ufw-%E9%98%B2%E7%81%AB%E5%A2%99/)

[如何在 Ubuntu 22.04 LTS 上启用/禁用防火墙](https://zh-cn.linuxcapable.com/how-to-enable-disable-firewall-on-ubuntu-22-04-lts/)
