---
title: 使用Apache Benchmark进行站点压力测试
date: 2017-05-27 11:11:18
tags:
- Apache Benchmark
- 压力测试
categories:
- Apache Benchmark
---
# 使用ab进行站点压力测试

## 压力测试相关概念

* 吞吐率（Requests per second）

    概念：服务器并发处理能力的量化描述，单位是reqs/s，指的是某个并发用户数下单位时间内处理的请求数。某个并发用户数下单位时间内能处理的最大请求数，称之为最大吞吐率。
    计算公式：总请求数 / 处理完成这些请求数所花费的时间，即
    Request per second = Complete requests / Time taken for tests

* 并发连接数（The number of concurrent connections）

    概念：某个时刻服务器所接受的请求数目，简单的讲，就是一个会话。

    并发用户数（The number of concurrent users，Concurrency Level）

    概念：要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。

* 用户平均请求等待时间（Time per request）

    计算公式：处理完成所有请求数所花费的时间/ （总请求数 / 并发用户数），即
    Time per request = Time taken for tests /（ Complete requests / Concurrency Level）

* 服务器平均请求等待时间（Time per request: across all concurrent requests）

    计算公式：处理完成所有请求数所花费的时间 / 总请求数，即
    Time taken for / testsComplete requests
    可以看到，它是吞吐率的倒数。
    同时，它也=用户平均请求等待时间/并发用户数，即
    Time per request / Concurrency Level

### 下载工具

[官网下载网址](https://www.apachelounge.com/download/ "下载链接") 

解压压缩包，进入到`httpd-2.4.25-win64-VC14\Apache24\bin`目录下，
打开命令行，输入命令

### 运行测试

``` ab
ab -n 100 -c 10 http://baidu.com/
```

其中－n表示请求数，－c表示并发数

### 测试结果

![测试结果](/img/ab测试结果.png)

### 完整测试报告

#### 服务器信息

    显示服务器为apache,域名，端口；

```ab
Server Software:        Apache
Server Hostname:        baidu.com
Server Port:            80
```

#### 文档信息

    所在位置“/”，文档的大小为338436 bytes（此为http响应的正文长度）

```ab
Document Path:          /
Document Length:        81 bytes
```

#### 重要信息

```ab
Concurrency Level:      10
Time taken for tests:   2.584 seconds
Complete requests:      100
Failed requests:        0
Total transferred:      38100 bytes
HTML transferred:       8100 bytes
Requests per second:    38.70 [#/sec] (mean)
Time per request:       258.374 [ms] (mean)
Time per request:       25.837 [ms] (mean, across all concurrent requests)
Transfer rate:          14.40 [Kbytes/sec] received
```

```ab
//并发请求数
Concurrency Level: 10

//整个测试持续的时间
Time taken for tests: 2.584 seconds

//完成的请求数
Complete requests: 100

//失败的请求数
Failed requests: 0

//整个场景中的网络传输量
Total transferred: 13701482 bytes

//整个场景中的HTML内容传输量
HTML transferred: 13197000 bytes

//吞吐率，大家最关心的指标之一，相当于 LR 中的每秒事务数，后面括号中的 mean 表示这是一个平均值
Requests per second: 38.70 [#/sec] (mean)

//用户平均请求等待时间，大家最关心的指标之二，相当于 LR 中的平均事务响应时间，后面括号中的 mean 表示这是一个平均值
Time per request: 258.374 [ms] (mean)

//服务器平均请求处理时间，大家最关心的指标之三
Time per request: 25.837 [ms] (mean, across all concurrent requests)

//平均每秒网络上的流量，可以帮助排除是否存在网络流量过大导致响应时间延长的问题
Transfer rate: 14.40 [Kbytes/sec] received
```

#### 网络上消耗的时间的分解

```ab
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       12   25  38.9     20     408
Processing:     8  223 120.3    196     591
Waiting:        8  142 124.9    120     574
Total:         29  248 125.7    217     609
```

#### 网络消耗时间

这段是每个请求处理时间的分布情况，50%的处理时间在217ms内，66%的处理时间在224ms内...，重要的是看90%的处理时间

```ab
Percentage of the requests served within a certain time (ms)
  50%    217
  66%    224
  75%    246
  80%    249
  90%    595
  95%    604
  98%    609
  99%    609
 100%    609 (longest request)
```

#### 登录的问题

    有时候进行压力测试需要用户登录，怎么办？
    请参考以下步骤：

    先用账户和密码登录后，用开发者工具找到标识这个会话的Cookie值（Session ID）记下来
    如果只用到一个Cookie，那么只需键入命令：
    ab －n 100 －C key＝value http://baidu.com/

    如果需要多个Cookie，就直接设Header：
    ab -n 100 -H “Cookie: Key1=Value1; Key2=Value2” http://baidu.com/

    同类型的压力测试工具还有：webbench、siege、http_load等

参考：

[http://www.cnblogs.com/lpfuture/p/5740831.html](http://www.cnblogs.com/lpfuture/p/5740831.html "参考1")