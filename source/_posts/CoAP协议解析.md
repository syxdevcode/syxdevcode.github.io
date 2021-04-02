---
title: CoAP协议解析
date: 2021-04-02 11:26:19
tags:
- 计算机基础
- 网络基础
- Linux网络
- UDP
- 网络协议
- Linux
- CentOS7
- MQTT协议
- CoAP协议
- IOT
- 物联网
- 嵌入式
categories:
- CoAP协议
---

## 介绍

&emsp;&emsp;The Constrained Application Protocol（CoAP,受限制的应用协议）是一种专用的Web传输协议，用于受约束的节点和受约束的(例如，低功率，有损)网络。

&emsp;&emsp;CoAP是一种应用层协议，它运行于 UDP协议之上。该协议旨在用于机器对机器（M2M）应用，例如智能能源和楼宇自动化。

![coap-stack.png](/img/coap-stack.png)

&emsp;&emsp;CoAP提供了应用程序端点之间的请求/响应交互模型，支持服务的资源发现，并包括Web的关键概念，例如URI和Internet媒体类型。CoAP旨在轻松与HTTP交互以与Web集成，同时满足诸如多播支持，非常低的开销以及在受限环境中的简单性等特殊要求。

&emsp;&emsp;互联网上的Web服务（Web API）的使用在大多数应用程序中已经无处不在，并且依赖于Web 的Representational State Transfer（REST，表述性状态传递）体系结构。

&emsp;&emsp;Constrained RESTful Environments（CoRE）的工作旨在以最合适的形式实现REST体系结构，以适用于最受约束的节点（例如RAM和ROM受限的8位微控制器）和网络（例如6LoWPAN）。诸如6LoWPAN之类的受约束的网络支持将IPv6数据包分段成小的链路层帧。但是，这会大大减少数据包交付概率。CoAP的一个设计目标是保持消息开销较小，从而限制了分段的需要。

&emsp;&emsp;CoAP的主要目标之一是针对这种受限环境的特殊要求设计通用的Web协议，尤其是考虑到能源，楼宇自动化以及其他机器对机器（M2M）应用程序。

&emsp;&emsp;CoAP的目标不是盲目地压缩HTTP，而是实现与HTTP通用但针对M2M应用程序进行了优化的REST的子集。尽管CoAP可用于将简单的HTTP接口重新生成更紧凑的协议，更重要的是，它还提供了M2M的功能，例如内置资源发现，多播支持和异步消息交换。CoAP协议非常小巧，最小的数据包仅为4字节。

&emsp;&emsp;该协议可以轻松转换为HTTP以与现有Web集成，同时满足特殊要求，例如多播支持，非常低的开销以及受约束环境和M2M应用程序的简便性。

## 特性

CoAP具有以下主要功能:

* 在受限条件下满足M2M要求的Web协议
* UDP [ RFC0768 ]绑定，具有可选的可靠性，支持单播和多播请求。
* 异步消息交换。
* 低的报头开销和解析复杂度。
* URI和内容类型支持。
* 简单的代理和缓存功能。
* 无状态HTTP映射，允许构建代理以提供通过HTTP统一方式或HTTP访问CoAP资源。
* 绑定到数据报传输层安全性（DTLS）的安全性。

## 消息格式

CoAP协议总是以 "头" 的形式出现在负载之前，而负载和CoAP头之间使用单字节0xFF分离。

![coap.png](/img/coap.png)

### 消息头（HEAD）

第一行是消息头，必须有，固定4个byte。

* Ver，Version：指示CoAP协议的版本号。2位无符号整数（2bit），取值1(01二进制)，其他值保留用于将来的版本。
* T，Type：报文类型，2位无符号整数（2bit），CoAP协议定了4种不同形式的报文，CON，NON，ACK，RST，全称：【Confirmable(0), Non-Confirmable, Acknowledgement(2)或者Reset(3)：。
* TKL，Token Length：CoAP标识符长度，4位无符号整数（4bit）。指示可变长度令牌字段的长度（0-8个字节）。长度9-15 保留，不得发送，并且必须作为消息格式错误进行处理。
* Code：功能码/响应码，8位无符号整数（8bit）。Code 在 CoAP 请求报文和响应报文中具有不同的表现形式，Code占一个字节，它被分成了两部分，前3位一部分，后5位一部分。
* Message ID：报文编号，16位无符号整数（16bit），用于检测消息重复并将确认/重置类型的消息与可确认/不可确认类型的消息进行匹配，每个消息都有一个ID ，重发的消息MID不变。
* Token：标识符具体内容（可选），通过 TKL 指定Token长度，用于将响应与请求匹配。 token值为0到8字节的序列。 (每条消息必须带有一个标记, 即使它的长度为零）。每个请求都带有一个客户端生成的token, 服务器在任何结果响应中都必须对其进行回应。token类似消息ID，用以标记消息的唯一性。token还是消息安全性的一个设置，使用全8字节的随机数，使伪造的报文无法获得验证通过。
* Option：报文选项，通过报文选项可设定CoAP主机，CoAP URI，CoAP请求参数和负载媒体类型等等。

1111 1111B(0xFF),CoAP报文和具体负载之间的分隔符。

### Code部分

* 0.00 Indicates an Empty message （空消息）
* 0.01-0.31 Indicates a request.（请求码）
* 1.00-1.31 Reserved(保留)
* 2.00-5.31 Indicates a response.（响应码）
* 6.00-7.31 Reserved(保留)

#### 请求

* 0.01：GET方法——用于获得某资源
* 0.02：POST方法——用于创建某资源
* 0.03：PUT方法——用于更新某资源
* 0.04：DELETE方法——用于删除某资源

### 响应

**响应码**

* 2.01：Created
* 2.02：Deleted
* 2.03：Valid
* 2.04：Changed
* 2.05：Content。类似于HTTP 200 OK

**Client Error 4.xx**

* 4.00：Bad Request 请求错误，服务器无法处理。类似于HTTP 400。
* 4.01：Unauthorized 没有范围权限。类似于HTTP 401。
* 4.02：Bad Option 请求中包含错误选项。
* 4.03：Forbidden 服务器拒绝请求。类似于HTTP 403。
* 4.04：Not Found 服务器找不到资源。类似于HTTP 404。
* 4.05：Method Not Allowed 非法请求方法。类似于HTTP 405。
* 4.06：Not Acceptable 请求选项和服务器生成内容选项不一致。类似于HTTP 406。
* 4.12：Precondition Failed 请求参数不足。类似于HTTP 412。
* 4.15：Unsuppor Conten-Type 请求中的媒体类型不被支持。类似于HTTP 415。

**Server Error 5.xx**

* 5.00：Internal Server Error 服务器内部错误。类似于HTTP 500。
* 5.01：Not Implemented 服务器无法支持请求内容。类似于HTTP 501。
* 5.02：Bad Gateway 服务器作为网关时，收到了一个错误的响应。类似于HTTP 502。
* 5.03：Service Unavailable 服务器过载或者维护停机。类似于HTTP 503。
* 5.04：Gateway Timeout 服务器作为网关时，执行请求时发生超时错误。类似于HTTP 504。
* 5.05：Proxying Not Supported 服务器不支持代理功能。

### Options部分

Options占用的字节数不定，如果包尾遇到 payload 标识符 0xff，则表示Options数据结束

Options类似于http协议中的Options，有 Content-Format、Uri-Path 之类的信息，格式解析与组包较为复杂，具体结构与其代表的数值如下：

![微信截图_20210402152447.png](/img/微信截图_20210402152447.png)

Option及编号的定义：

| No.    | Name | Format| Length | Default| 
| :----- | :---- | :----: | :---- |:---- |
| 1	| If-Match	| opaque| 	0-8| 	(none)| 
| 3	| Uri-Host| 	string| 	1-255	| (see note 1)| 
| 4	| ETag| 	opaque| 	1-8	| (none)| 
| 5	| If-None-Match| 	empty| 	0	| (none)| 
| 7	| Uri-Port| 	uint| 	0-2| 	(see note 1)| 
| 8	| Location-Path	| string| 	0-255 | 	(none)| 
| 11| 	Uri-Path| 	string| 	0-255	| (none)| 
| 12| 	Content-Format| 	uint| 	0-2| 	(none)| 
| 14| 	Max-Age| 	uint| 	0-4	| 60| 
| 15| 	Uri-Query| 	string| 	0-255| 	(none)| 
| 17| 	Accept| 	uint| 	0-2| 	(none)| 
| 20| 	Location-Query| 	string| 	0-255| 	(none)| 
| 28| 	Size2	| uint| 	0-4	| (none)| 
| 35| 	Proxy-Uri| 	string| 	1-1034| 	(none)| 
| 39| 	Proxy-Scheme| 	string| 	1-255	| (none)| 
| 60| 	Size1	| uint| 	0-4	| (none)| 

Content-Formats参数的具体数值：

| Media type | Id.| 
| :----- | :---- |
| text/plain;charset=utf-8| 	0| 
| application/link-format| 	40| 
| application/xml| 	41| 
| application/octet-stream| 	42| 
| application/exi	| 47| 
| application/json	| 50| 
| application/cbor	| 60| 

#### Option Delta

`Option Delta` 占用4位（0.5字节）

`Option Delta` 代表Option的类型，该值代表了上表中Option类型的代码值与上一个Option代码值之间的差值
（如果该Option为第一个Option，则直接表达该Option的 `Option Delta`）

由于 `Option Delta` 只有4位，最大只能表达15，为了解决这个问题，coap协议有着如下规定：

* 当 `Option Delta <=12` 时：`Option Delta` 位为实际的 `Option Delta` 值;
* 当 `Option Delta <269` 时：`Option Delta` 位填入 `13` ；并且在后面的 `Option Delta(extended)` 位会占用 `1字节`，并且填入的数为实际 `Option Delta` 值减去 `13`;
* 当 `Option Delta <65804` 时： `Option Delta` 位填入 `14` ；并且在后面的 `Option Delta(extended)` 位会占用 `2字节` ，并且填入的数为实际 `Option Delta` 值减去 `269`。

特别注意，填入的 `Option Delta` 值不可能为 `15（0x0f）`，当遇到 `15（0x0f）` 时，该包无效。

#### Option Length

`Option Length` 占用4位（0.5字节）

`Option Length` 代表该option所包含数据（value）的长度，该值的表示方法类似于 `Option Delta`，如下：

* 当 `Option Length <=12` 时：`Option Length` 位为实际的 `Option Length` 值;
* 当 `Option Length <269` 时：`Option Length` 位填入`13`；并且在后面的 `Option Length(extended)` 位会占用 `1字节` ，并且填入的数为实际 `Option Length` 值减去 `13`;
* 当 `Option Length <65804` 时：`Option Length` 位填入 `14`；并且在后面的 `Option Length(extended)` 位会占用 `2字节`，并且填入的数为实际 `Option Length` 值减去 `269`;

填入的 `Option Length` 值不可能为 `15（0x0f）`，当遇到 `15（0x0f）` 时，该包无效。

#### 多个option的情况

当有多个option时，这些option必须是按option代码值（No.）的顺序**从小到大**排列的，不然会导致Option Delta的值出错。
每个部分的option各自按格式组包，最后按顺序拼到一起，并入字节流中。

## 负载内容（Payload）

Payload 占用字节数不定

当Payload不存在时，数据包末尾不能加上 0xff 的分隔符

如果存在0xff的分隔符，则分隔符后的数据便是 Payload。


## COAP的安全性

COAP的安全性是用DTLS加密实现的。DTLS的实现需要的资源和带宽较多，如果是资源非常少的终端和极有限的带宽下可能会跑不起来。DTLS仅仅在单播情况下适用。

![微信截图_20210402141929.png](/img/微信截图_20210402141929.png)

## CoAP客户端

* [CoAP-CLI](https://www.npmjs.com/package/coap-cli): CoAP-CLI是CoAP的命令行界面，基于node.js和[node-coap](https://www.npmjs.com/package/coap-cli)所构建。
* [CoAP Shell](https://github.com/tzolov/coap-shell) 提供用于与CoAP协议交互的命令行界面。它支持coap:和coaps模式(例如UDP和DTLS)。CoAP Shell建立在Spring Shell, Californium(Cf)和Scandium(Sc)项目之上。它是一个SpringBoot应用程序,它内置于单个可自我执行的jar中，并且可以在任何Java8+环境中运行。

### 使用CoAP Shell

```sh
git clone https://github.com/sanshengshui/coap-shell
cd coap-shell
mvn clean install
cd target
java -jar coap-shell-1.1.2-SNAPSHOT.jar

# 可以使用 coap://californium.eclipse.org/或coap://coap.me
connect coap://coap.me
```

交互：

```ps1
# 发现COAP资源
coap://coap.me:>discover

# get
coap://coap.me:>get /hello
coap://coap.me:>get /sink

# 删除
coap://coap.me:>delete /sink

# PUT
coap://coap.me:>put /sink --payload 'Hi From IoT' --format text/plain

# POST
coap://coap.me:>post /sink --payload 'testing for post data' --format text/plain
```

```ps1
PS E:\git\coap-shell> cd target
PS E:\git\coap-shell\target> java -jar coap-shell-1.1.2-SNAPSHOT.jar

  _____     ___   ___     ______       ____
 / ___/__  / _ | / _ \   / __/ /  ___ / / /
/ /__/ _ \/ __ |/ ___/  _\ \/ _ \/ -_) / /
\___/\___/_/ |_/_/     /___/_//_/\__/_/_/
CoAP Shell (v1.1.2-SNAPSHOT)
For assistance hit TAB or type "help".

server-unknown:>connect coap://coap.me
available
coap://coap.me:>discover
┌──────────────────────────────┬────────────────────────┬──────────────────────────┬────────────┬───────┬─────────────┐
│Path [href]                   │Resource Type [rt]      │Content Type [ct]         │Interface   │Size   │Observable   │
│                              │                        │                          │[if]        │[sz]   │[obs]        │
├──────────────────────────────┼────────────────────────┼──────────────────────────┼────────────┼───────┼─────────────┤
│/123412341234123412341234     │123412341234123412341234│text/plain (0)            │            │       │             │
│/3                            │3                       │application/json (50)     │            │       │             │
│/4                            │4                       │application/json (50)     │            │       │             │
│/5                            │5                       │application/json (50)     │            │       │             │
│/bl%C3%A5b%C3%A6rsyltet%C3%B8y│blåbærsyltetøy          │text/plain (0)            │            │       │             │
│/broken                       │Type2, Type1            │text/plain (0)            │If2, If1    │       │             │
│/create1                      │create1                 │text/plain (0)            │            │       │             │
│/hello                        │Type1                   │text/plain (0)            │If1         │       │             │
│/large                        │Type1, Type2            │text/plain (0)            │If2         │1700   │             │
│/large-create                 │large-create            │text/plain (0)            │            │       │             │
│/large-update                 │large-update            │text/plain (0)            │            │       │             │
│/location-query               │location-query          │text/plain (0)            │            │       │             │
│/location1                    │location1               │application/link-format   │            │       │             │
│                              │                        │(40)                      │            │       │             │
│/multi-format                 │multi-format            │text/plain (0)            │            │       │             │
│/path                         │path                    │application/link-format   │            │       │             │
│                              │                        │(40)                      │            │       │             │
│/query                        │query                   │text/plain (0)            │            │       │             │
│/secret                       │secret                  │text/plain (0)            │            │       │             │
│/seg1                         │seg1                    │application/link-format   │            │       │             │
│                              │                        │(40)                      │            │       │             │
│/separate                     │separate                │text/plain (0)            │            │       │             │
│/sink                         │sink                    │text/plain (0)            │            │       │             │
│/test                         │test                    │text/plain (0)            │            │       │             │
│/validate                     │validate                │text/plain (0)            │            │       │             │
│/weird33                      │weird33                 │text/plain (0)            │            │       │             │
│/weird333                     │weird333                │text/plain (0)            │            │       │             │
│/weird3333                    │weird3333               │text/plain (0)            │            │       │             │
│/weird33333                   │weird33333              │text/plain (0)            │            │       │             │
│/weird44                      │weird44                 │text/plain (0)            │            │       │             │
│/weird55                      │weird55                 │text/plain (0)            │            │       │             │
└──────────────────────────────┴────────────────────────┴──────────────────────────┴────────────┴───────┴─────────────┘

coap://coap.me:>get /hello
----------------------------------- Response -----------------------------------
GET coap://coap.me/hello
MID: 2484, Type: ACK, Token: F83EC22681D5777D, RTT: 307ms
Options: {"Content-Format":"text/plain"}
Status : 205-Reset Content, Payload: 5B
................................... Payload ....................................
world
--------------------------------------------------------------------------------
coap://coap.me:>get /sink
----------------------------------- Response -----------------------------------
GET coap://coap.me/sink
MID: 2485, Type: ACK, Token: E06976CDA3B63624, RTT: 307ms
Options: {"ETag":0xc881f5f43b670f49, "Content-Format":"text/plain"}
Status : 205-Reset Content, Payload: 16B
................................... Payload ....................................
you put here: 23
--------------------------------------------------------------------------------
coap://coap.me:>delete /sink
----------------------------------- Response -----------------------------------
DELETE coap://coap.me/sink
MID: 2486, Type: ACK, Token: A80637C001DAC74F, RTT: 308ms
Options: {"Content-Format":"text/plain"}
Status : 202-Accepted, Payload: 9B
................................... Payload ....................................
DELETE OK
--------------------------------------------------------------------------------
coap://coap.me:>get /sink
----------------------------------- Response -----------------------------------
GET coap://coap.me/sink
MID: 2487, Type: ACK, Token: 54667D264357A5DD, RTT: 307ms
Options: {"Content-Format":"text/plain"}
Status : 205-Reset Content, Payload: 13B
................................... Payload ....................................
I was deleted
--------------------------------------------------------------------------------
coap://coap.me:>put /sink --payload 'Hi From IoT' --format text/plain
----------------------------------- Response -----------------------------------
PUT coap://coap.me/sink
MID: 2488, Type: ACK, Token: C45F0869EF238705, RTT: 307ms
Options: {"Content-Format":"text/plain"}
Status : 204-No Content, Payload: 6B
................................... Payload ....................................
PUT OK
--------------------------------------------------------------------------------
coap://coap.me:>get /sink
----------------------------------- Response -----------------------------------
GET coap://coap.me/sink
MID: 2489, Type: ACK, Token: 54A03150A4CD86F5, RTT: 306ms
Options: {"ETag":0xc91c846032a1ac9f, "Content-Format":"text/plain"}
Status : 205-Reset Content, Payload: 25B
................................... Payload ....................................
you put here: Hi From IoT
--------------------------------------------------------------------------------
coap://coap.me:>delete /sink
----------------------------------- Response -----------------------------------
DELETE coap://coap.me/sink
MID: 2490, Type: ACK, Token: 1C3786AFE682101E, RTT: 308ms
Options: {"Content-Format":"text/plain"}
Status : 202-Accepted, Payload: 9B
................................... Payload ....................................
DELETE OK
--------------------------------------------------------------------------------
coap://coap.me:>post /sink --payload 'testing for post data' --format text/plain
----------------------------------- Response -----------------------------------
POST coap://coap.me/sink
MID: 2491, Type: ACK, Token: E87513DE0775F3A2, RTT: 306ms
Options: {"Location-Path":["location1","location2","location3"], "Content-Format":"text/plain"}
Status : 201-Created, Payload: 7B
................................... Payload ....................................
POST OK
--------------------------------------------------------------------------------
coap://coap.me:>get /sink
----------------------------------- Response -----------------------------------
GET coap://coap.me/sink
MID: 2492, Type: ACK, Token: 54A44124515C48D0, RTT: 306ms
Options: {"ETag":0xf97973ea26db6781, "Content-Format":"text/plain"}
Status : 205-Reset Content, Payload: 54B
................................... Payload ....................................
I was deleted, and you put here: testing for post data
--------------------------------------------------------------------------------
```

参考：

[CoAP学习笔记——CoAP格式详解](https://blog.csdn.net/xukai871105/article/details/45167069)

[抓住CoAP协议的'心'](https://blog.mushuwei.cn/2020/05/07/%E6%8A%93%E4%BD%8FCoAP%E5%8D%8F%E8%AE%AE%E7%9A%84-%E5%BF%83/)

[coap组包格式的简单解析](https://www.chenxublog.com/2018/09/28/coap-pack-by-hand.html)

[物联网协议Coap协议介绍](https://baijiahao.baidu.com/s?id=1609055547851599818&wfr=spider&for=pc)