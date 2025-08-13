---
title: JMeter性能压测简介
date: 2021-01-06 12:45:41
tags:
- JMeter
- 性能测试
categories:
- JMeter
---

## 线程组

![微信截图_20210106142353.png](/img/微信截图_20210106142353.png)

线程组参数详解： 
* 线程数：虚拟用户数。一个虚拟用户占用一个进程或线程。设置多少虚拟用户数在这里也就是设置多少个线程数。 
* Ramp-Up Period(in seconds)准备时长：设置的虚拟用户数需要多长时间全部启动。如果线程数为10，准备时长为2，那么需要2秒钟启动10个线程，也就是每秒钟启动5个线程。 
* 循环次数：每个线程发送请求的次数。如果线程数为10，循环次数为100，那么每个线程发送100次请求。总请求数为10*100=1000 。如果勾选了“永远”，那么所有线程会一直发送请求，一到选择停止运行脚本。 
* Delay Thread creation until needed：直到需要时延迟线程的创建。 
* 调度器：设置线程组启动的开始时间和结束时间(配置调度器时，需要勾选循环次数为永远) 
  持续时间（秒）：测试持续时间，会覆盖结束时间 
  启动延迟（秒）：测试延迟启动时间，会覆盖启动时间 

## 聚合报告参数详解

* Label：每个 JMeter 的 element（例如 HTTP Request）都有一个 Name 属性，这里显示的就是 Name 属性的值 
* #Samples：请求数——表示这次测试中一共发出了多少个请求，如果模拟10个用户，每个用户迭代10次，那么这里显示100 
* Average：平均响应时间——默认情况下是单个 Request 的平均响应时间，当使用了 Transaction Controller 时，以Transaction 为单位显示平均响应时间 
* Median：中位数，也就是 50％ 用户的响应时间 
* 90% Line：90％ 用户的响应时间 
* Min：最小响应时间 
* Max：最大响应时间 
* Error%：错误率——错误请求数/请求总数 
* Throughput：吞吐量——默认情况下表示每秒完成的请求数（Request per Second），当使用了 Transaction Controller 时，也可以表示类似 LoadRunner 的 Transaction per Second 数 
* KB/Sec：每秒从服务器端接收到的数据量，相当于LoadRunner中的Throughput/Sec

参考:

[JMeter性能测试，完整入门篇](https://blog.csdn.net/u012111923/article/details/80705141)