---
title: 缓存击穿，失效以及热点key
date: 2018-06-12 14:41:31
tags:
- 分布式
- 缓存
categories: 
- 缓存
---


## 缓存击穿

缓存一般是Key，value方式存在，当某一个Key不存在时会查询数据库，假如这个Key，一直不存在，则会频繁的请求数据库，对数据库造成访问压力。

### 1、使用互斥锁

在根据key获得的value值为空时，先锁上，再从数据库加载，加载完毕，释放锁。若其他线程发现获取锁失败，则睡眠50ms后重试。

当通过某一个key去查询数据的时候，如果对应在数据库中的数据都不存在（不管是数据不存在，还是系统故障），我们将此key对应的value设置为一个默认的值，比如“NULL”，并设置一个缓存的失效时间，它的过期时间会很短，最长不超过五分钟。这时在缓存失效之前，所有通过此key的访问都被缓存挡住了。后面如果此key对应的数据在DB中存在时，缓存失效之后，通过此key再去访问数据，就能拿到新的value了。

至于锁的类型，单机环境用并发包的Lock类型就行，集群环境则使用分布式锁( redis的setnx)

** 优点**

思路简单
保证一致性

** 缺点**

代码复杂度增大
存在死锁的风险

** SETNX语法**

SETNX key value

将 key 的值设为 value ，当且仅当 key 不存在。

若给定的 key 已经存在，则 SETNX 不做任何动作。

SETNX 是『SET if Not eXists』(如果不存在，则 SET)的简写。

可用版本：>= 1.0.0

时间复杂度： O(1)

返回值： 设置成功，返回 1。设置失败，返回 0 。

```redis
172.19.0.2:6380> EXISTS job
(integer) 0
172.19.0.2:6380> setnx job "programmer"
(integer) 1
172.19.0.2:6380> setnx job "code-farmer"
(integer) 0
172.19.0.2:6380> get job
"programmer"
```

.net 代码

参考：[http://www.cnblogs.com/yaoshangjin/p/7456378.html](http://www.cnblogs.com/yaoshangjin/p/7456378.html)

```csharp
/// <summary>
/// 分布式锁
/// </summary>
/// <param name="resource">The thing we are locking on</param>
/// <param name="expirationTime">The time after which the lock will automatically be expired by Redis</param>
/// <param name="action">Action to be performed with locking</param>
/// <returns>True if lock was acquired and action was performed; otherwise false</returns>
public bool PerformActionWithLock(string resource, TimeSpan expirationTime, Action action)
{
    RedisValue lockToken = Guid.NewGuid().ToString(); //Environment.MachineName;
    RedisKey locKey = resource;
    var db = GetDatabase();
    if (db.LockTake(locKey, lockToken, expirationTime))
    {
        try
        {
            action();
        }
        finally
        {
            db.LockRelease(locKey, lockToken);
        }
        return true;
    }
    else
    {
        return false;
    }
}
```

测试代码：

``` csharp
[Test]
public void TestLock()
{
    RedisKey _lockKey = "lock_key";
    var _lockDuration = TimeSpan.FromSeconds(30);
    var mainTask = new Task(() => _redisMSConnectionWrapper.PerformActionWithLock(_lockKey, _lockDuration, () =>
    {
        Console.WriteLine("1.执行锁内任务-开始");
        Thread.Sleep(10000);
        Console.WriteLine("1.执行锁内任务-结束");
    }));
    mainTask.Start();
    Thread.Sleep(1000);
    var subOk = true;
    while (subOk)
    {
        _redisMSConnectionWrapper.PerformActionWithLock(_lockKey, _lockDuration, () =>
        {
            Console.WriteLine("2.执行锁内任务-开始");
            Console.WriteLine("2.执行锁内任务-结束");
            subOk = false;
        });
        Thread.Sleep(1000);
    }
    Console.WriteLine("结束");
}
```

### 2、异步构建缓存

或者叫"提前"使用互斥锁

在value内部设置1个超时值(timeout1), timeout1比实际的memcache timeout(timeout2)小。当从cache读取到timeout1发现它已经过期时候，马上延长timeout1并重新设置到cache。然后再从数据库加载数据并设置到cache中。

** 优点**

性价最佳，用户无需等待

** 缺点**

无法保证缓存一致性

### 3、布隆过滤器

布隆过滤器的巨大用处就是，能够迅速判断一个元素是否在一个集合中。因此他有如下三个使用场景:

1.网页爬虫对URL的去重，避免爬取相同的URL地址
2.反垃圾邮件，从数十亿个垃圾邮件列表中判断某邮箱是否垃圾邮箱（同理，垃圾短信）
3.缓存击穿，将已存在的缓存放到布隆过滤器中，当黑客访问不存在的缓存时迅速返回避免缓存及DB挂掉。

** 布隆过滤器的原理**

其内部维护一个全为0的bit数组，需要说明的是，布隆过滤器有一个误判率的概念，误判率越低，则数组越长，所占空间越大。误判率越高则数组越小，所占的空间越小。

** 优点**

思路简单
保证一致性
性能强

** 缺点**

代码复杂度增大
需要另外维护一个集合来存放缓存的Key
布隆过滤器不支持删值操作

## 缓存雪崩（缓存失效）

在高并发的环境下，缓存集中在一段时间内大量失效（例如设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效），此时有多个进程就会去同时去查询DB，然后再去同时设置缓存。DB访问量会瞬间增大，造成了缓存雪崩。

解决方法：

1, 在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待，参考上文：1、使用互斥锁。
2, 可以通过缓存reload机制，预先去更新缓存，再即将发生大并发访问前手动触发加载缓存。
3, 不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀,例如在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。
4, 做二级缓存，或者双缓存策略。A1为原始缓存，A2为拷贝缓存，A1失效时，可以访问A2，A1缓存失效时间设置为短期，A2设置为长期。

## 热点key问题

(1) 这个key是一个热点key（例如一个重要的新闻，一个热门的八卦新闻等等），所以这种key访问量可能非常大。

(2) 缓存的构建是需要一定时间的。（可能是一个复杂计算，例如复杂的sql、多次IO、多个依赖(各种接口)等等）

于是就会出现一个致命问题：在缓存失效的瞬间，有大量线程来构建缓存，或者由于设计问题，造成缓存击穿，造成后端负载加大，甚至可能会让系统崩溃。

解决方法：

1，使用互斥锁(mutex key):这种解决方案思路比较简单，就是只让一个线程构建缓存，其他线程等待构建缓存的线程执行完，重新从缓存获取数据就可以了，参考上文：1、使用互斥锁。

2，"提前"使用互斥锁(mutex key)：在value内部设置1个超时值(timeout1), timeout1比实际的memcache timeout(timeout2)小。当从cache读取到timeout1发现它已经过期时候，马上延长timeout1并重新设置到cache。然后再从数据库加载数据并设置到cache中。参考上文：2、异步构建缓存

3，"永远不过期"：

这里的“永远不过期”包含两层意思：

(1) 从redis上看，确实没有设置过期时间，这就保证了，不会出现热点key过期问题，也就是“物理”不过期。

(2) 从功能上看，如果不过期，那不就成静态的了吗？所以我们把过期时间存在key对应的value里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建，也就是“逻辑”过期

参考：

[分布式之缓存击穿](http://www.cnblogs.com/rjzheng/p/8908073.html)

[缓存雪崩、缓存穿透、热点Key解决方案和分析](https://blog.csdn.net/wang0112233/article/details/79558612)

[缓存穿透，缓存击穿，缓存雪崩解决方案分析](https://blog.csdn.net/zeb_perfect/article/details/54135506)

[缓存击穿、失效以及热点key问题](https://www.jianshu.com/p/d5a3668d4dad)