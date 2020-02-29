---
title: Consul的分布式锁DotNet实现
date: 2018-06-13 10:23:53
tags:
- 分布式
- Consul
categories: 
- Consul
---
# Consul的分布式锁dotnet实现

## 分布式锁实现

基于Consul的分布式锁主要利用Key/Value存储API中的acquire和release操作来实现。acquire和release操作是类似Check-And-Set的操作：

acquire操作只有当锁不存在持有者时才会返回true，并且set设置的Value值，同时执行操作的session会持有对该Key的锁，否则就返回false

release操作则是使用指定的session来释放某个Key的锁，如果指定的session无效，那么会返回false，否则就会set设置Value值，并返回true

具体实现中主要使用了这几个Key/Value的API：

create session：[https://www.consul.io/api/session.html#session_create](https://www.consul.io/api/session.html#session_create)

delete session：[https://www.consul.io/api/session.html#delete-session](https://www.consul.io/api/session.html#delete-session)

KV acquire/release：[https://www.consul.io/api/kv.html#create-update-key](https://www.consul.io/api/kv.html#create-update-key)

## 基本流程

![consul-lock.png](/img/consul-lock.png)

以上内容引用自：[http://blog.didispace.com/spring-cloud-consul-lock-and-semphore/](http://blog.didispace.com/spring-cloud-consul-lock-and-semphore/)

代码参考：

[https://github.com/PlayFab/consuldotnet](https://github.com/PlayFab/consuldotnet)

参考：

[https://github.com/PlayFab/consuldotnet](https://github.com/PlayFab/consuldotnet)

[https://www.consul.io/docs/guides/leader-election.html](https://www.consul.io/docs/guides/leader-election.html)

[https://www.consul.io/api/kv.html](https://www.consul.io/api/kv.html)

[基于Consul的分布式锁实现](http://blog.didispace.com/spring-cloud-consul-lock-and-semphore/)