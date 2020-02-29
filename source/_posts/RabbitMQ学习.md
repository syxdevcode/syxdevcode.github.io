---
title: RabbitMQ学习
date: 2018-07-11 16:24:34
tags:
- 消息队列
- RabbitMQ
- DotNet
- CSharp
- DotNetCore
categories: 
- RabbitMQ
---
# RabbitMQ学习

## RabbitMQ 简介

RabbitMQ——Rabbit Message Queue的简写，是一个由erlang开发，基于AMQP（Advanced Message Queue Protocol）协议的开源实现。RabbitMQ提供了可靠的消息机制、跟踪机制和灵活的消息路由，支持消息集群和分布式部署，适用于排队算法、秒杀活动、消息分发、异步处理、数据同步、处理耗时任务、CQRS等应用场景。是当前最主流的消息中间件之一。

RabbitMQ的官网：[http://www.rabbitmq.com](http://www.rabbitmq.com)

![435188-20180605151314266-1010797270.png](/img/435188-20180605151314266-1010797270.png)

RabbitMQ 投递过程:

* ** 1.** 客户端连接到消息队列服务器，打开一个 channel。
* ** 2.** 客户端声明一个 exchange，并设置相关属性。
* ** 3.** 客户端声明一个 queue，并设置相关属性。
* ** 4.** 客户端使用 routing key，在 exchange 和 queue 之间建立好绑定关系。
* ** 5.** 客户端Producer投递消息到 exchange。
* ** 6.** 客户端Consumer从指定的 queue 中消费信息。

## AMQP 简介

AMQP，是应用层协议的一个开放标准，为面向消息的中间件设计。消息中间件主要用于组件之间的解耦，消息的发送者无需知道消息使用者的存在，同样，消息使用者也不用知道发送者的存在。AMQP的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。

## RabbitMQ 概念

* ** Exchange** : 消息交换机，它指定消息按什么规则，路由到哪个队列
* ** Queue** : 消息队列，每个消息都会被投入到一个或多个队列
* ** Routing Key** : 路由关键字，exchange根据这个关键字进行消息投递。
* ** Binding Key** : 绑定，它的作用就是把 exchange 和 queue 按照路由规则绑定起来
* ** Vhost** : 虚拟主机，可以开设多个 vhost，用作不同用户的权限分离
* ** Producer** : 消息生产者，投递消息的程序，简写 ：P
* ** Consumer** : 消息消费者，接受消息的程序，简写 : C
* ** Broker** ：消息队列的服务器实体。
* ** Channel** : 消息通道，在客户端的每个连接里，可建立多个 channel，每个 channel 代表一个会话任务

## 消息发送原理

应用程序和Rabbit Server之间会创建一个TCP连接，一旦TCP打开，并通过了认证，认证就是你试图连接Rabbit之前发送的Rabbit服务器连接信息和用户名和密码，有点像程序连接数据库，一旦认证通过你的应用程序和Rabbit就创建了一条AMQP信道（Channel）。

信道是创建在“真实”TCP上的虚拟连接，AMQP命令都是通过信道发送出去的，每个信道都会有一个唯一的ID，不论是发布消息，订阅队列或者介绍消息都是通过信道完成的。

为什么不通过TCP直接发送命令？对于操作系统来说创建和销毁TCP会话是非常昂贵的开销，假设高峰期每秒有成千上万条连接，每个连接都要创建一条TCP会话，这就造成了TCP连接的巨大浪费，而且操作系统每秒能创建的TCP也是有限的，因此很快就会遇到系统瓶颈。

如果我们每个请求都使用一条TCP连接，既满足了性能的需要，又能确保每个连接的私密性，这就是引入信道概念的原因。

## 使用 RabbitMQ

在.NET中使用RabbitMQ需要下载RabbitMQ的客户端程序集.

```bash
Install-Package RabbitMQ.Client -Version 5.1.0
```

[https://www.nuget.org/packages/RabbitMQ.Client/](https://www.nuget.org/packages/RabbitMQ.Client/)

### 工作队列

工作队列（work queues, 又称任务队列Task Queues）的主要思想是为了避免立即执行并等待一些占用大量资源、时间的操作完成。而是把任务（Task）当作消息发送到队列中，稍后处理。一个运行在后台的工作者（worker）进程就会取出任务然后处理。当运行多个工作者（workers）时，任务会在它们之间共享。

这个在网络应用中非常有用，它可以在短暂的HTTP请求中处理一些复杂的任务。在一些实时性要求不太高的地方，我们可以处理完主要操作之后，以消息的方式来处理其他的不紧要的操作，比如写日志等等。

#### 发送/接收消息

分别创建两个控制台项目Send、Receive。

安装 `RabbitMQ.Client`

注意：消息接收端和发送端的队列名称（queue）必须保持一致

![2799767-a5e45f97bec36c8a.png](/img/2799767-a5e45f97bec36c8a.png)

源代码：[https://github.com/syxdevcode/RabbitMqDemo](https://github.com/syxdevcode/RabbitMqDemo)

Send逻辑代码：

```csharp
namespace RabbitMqDemo.Send
{
    class Program
    {
        static void Main(string[] args)
        {
            //ServicePointManager.ServerCertificateValidationCallback += (sender, cert, chain, sslPolicyErrors) => true;
            ConnectionFactory factory = new ConnectionFactory()
            {
                UserName = "admin",
                Password = "admin",
                AutomaticRecoveryEnabled = true,
                Ssl = new SslOption()
                {
                    CertPath = @"E:\git\RabbitMqDemo\RabbitMqDemo.Send\server.pfx",
                    CertPassphrase = "123123",
                    AcceptablePolicyErrors = SslPolicyErrors.RemoteCertificateNameMismatch |
                                                SslPolicyErrors.RemoteCertificateChainErrors,
                    Enabled = true
                },
                //AuthMechanisms = new AuthMechanismFactory[] { new ExternalMechanismFactory() },
                RequestedHeartbeat = 60,
                Port = 5673,
                TopologyRecoveryEnabled = true
            };

            //第一步：创建connection 
            using (var connection = factory.CreateConnection(new string[1] { "192.168.0.115" }))
            {
                //第二步：创建一个channel信道
                using (var channel = connection.CreateModel())
                {
                    //第三步：申明队列 durable:是否持久化
                    var result = channel.QueueDeclare(queue: "test1", durable: true, exclusive: false, autoDelete: false, arguments: null);

                    for (int i = 0; i < int.MaxValue; i++)
                    {
                        //构建byte消息数据包
                        var body = Encoding.UTF8.GetBytes(i.ToString() + "test");
                        channel.BasicPublish(exchange: "", routingKey: "test1", basicProperties: null, body: body);
                        Console.WriteLine("{0} 推送成功", i);
                        Thread.Sleep(500);
                    }
                }
            }
            Console.Read();
        }
    }
}
```

Receive逻辑代码：

```csharp
static void Main(string[] args)
        {
            //1.实例化连接工厂
            ConnectionFactory factory = new ConnectionFactory()
            {
                UserName = "admin",
                Password = "admin",
                AutomaticRecoveryEnabled = true,
                Ssl = new SslOption()
                {
                    CertPath = @"E:\git\RabbitMqDemo\RabbitMqDemo.Receive\server.pfx",
                    CertPassphrase = "123123",
                    AcceptablePolicyErrors = SslPolicyErrors.RemoteCertificateNameMismatch |
                                                SslPolicyErrors.RemoteCertificateChainErrors,
                    Enabled = true
                },
                //AuthMechanisms = new AuthMechanismFactory[] { new ExternalMechanismFactory() },
                RequestedHeartbeat = 60,
                Port = 5673,
                TopologyRecoveryEnabled = true
            };

            //2. 建立连接
            using (var connection = factory.CreateConnection(new string[1] { "192.168.0.115" }))
            {
                //3. 创建信道
                using (var channel = connection.CreateModel())
                {
                    //4. 申明队列
                    channel.QueueDeclare(queue: "test1", durable: true, exclusive: false, autoDelete: false, arguments: null);
                    //5. 构造消费者实例
                    var consumer = new EventingBasicConsumer(channel);
                    //6. 绑定消息接收后的事件委托
                    consumer.Received += (model, ea) =>
                    {
                        var message = Encoding.UTF8.GetString(ea.Body);
                        Console.WriteLine(" [x] Received {0}", message);
                        Thread.Sleep(500);//模拟耗时
                        Console.WriteLine(" [x] Done");
                    };
                    //7. 启动消费者
                    channel.BasicConsume(queue: "test1", autoAck: true, consumer: consumer);
                    Console.WriteLine(" Press [enter] to exit.");
                    Console.ReadLine();
                }
            }
        }
```

#### 轮询分发

![2799767-283ced13913a0aac.png](/img/2799767-283ced13913a0aac.png)

我们先启动两个接收端，等待消息接收，再启动一个发送端进行消息发送。

![QQ截图20180712135714.png](/img/QQ截图20180712135714.png)

从图中可知，发送的信息，被两个消息接收端按顺序循环分配。
默认情况下，RabbitMQ将按顺序将每条消息发送给下一个消费者。平均每个消费者将获得相同数量的消息。这种分发消息的方式叫做** 循环（round-robin）**。

### 消息确认

当处理一个比较耗时得任务的时候，也许想知道消费者（consumers）是否运行到一半就挂掉。在当前的代码中，当RabbitMQ将消息发送给消费者（consumers）之后，马上就会将该消息从队列中移除。此时，如果把处理这个消息的工作者（worker）停掉，正在处理的这条消息就会丢失。同时，所有发送到这个工作者的还没有处理的消息都会丢失。

我们不想丢失任何任务消息。如果一个工作者（worker）挂掉了，我们希望该消息会重新发送给其他的工作者（worker）。

为了防止消息丢失，RabbitMQ提供了** 消息响应（message acknowledgments）**机制。消费者会通过一个ack（响应），告诉RabbitMQ已经收到并处理了某条消息，然后RabbitMQ才会释放并删除这条消息。

如果消费者（consumer）挂掉了，没有发送响应，RabbitMQ就会认为消息没有被完全处理，然后重新发送给其他消费者（consumer）。这样，即使工作者（workers）偶尔的挂掉，也不会丢失消息。

消息是没有超时这个概念的；当工作者与它断开连的时候，RabbitMQ会重新发送消息。这样在处理一个耗时非常长的消息任务的时候就不会出问题了。

```csharp
namespace RabbitMqDemo.Receive
{
    class Program
    {
        static void Main(string[] args)
        {
            //1.实例化连接工厂
            // 加证书
            //ConnectionFactory factory = new ConnectionFactory()
            //{
            //    UserName = "admin",
            //    Password = "admin",
            //    Ssl = new SslOption()
            //    {
            //        CertPath = @"E:\git\RabbitMqDemo\RabbitMqDemo.Receive\server.pfx",
            //        CertPassphrase = "123123",

            //        AcceptablePolicyErrors = SslPolicyErrors.RemoteCertificateNameMismatch |
            //                                    SslPolicyErrors.RemoteCertificateChainErrors,
            //        Enabled = true
            //    },
            //    //AuthMechanisms = new AuthMechanismFactory[] { new ExternalMechanismFactory() },
            //    RequestedHeartbeat = 60,
            //    Port = 5673
            //};

            ConnectionFactory factory = new ConnectionFactory()
            {
                UserName = "admin",
                Password = "admin",
                AutomaticRecoveryEnabled = true,
                Port = 5672,
                TopologyRecoveryEnabled = true
            };

            //2. 建立连接
            using (var connection = factory.CreateConnection(new string[1] { "192.168.0.115" }))
            {
                //3. 创建信道
                using (var channel = connection.CreateModel())
                {
                    //4. 申明队列
                    channel.QueueDeclare(queue: "test1", durable: true, exclusive: false, autoDelete: false, arguments: null);
                    channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
                    //5. 构造消费者实例
                    var consumer = new EventingBasicConsumer(channel);
                    //6. 绑定消息接收后的事件委托
                    consumer.Received += (model, ea) =>
                    {
                        var message = Encoding.UTF8.GetString(ea.Body);
                        Console.WriteLine(" [x] Received {0}", message);
                        // TODO 注：耗时过长（测试使用5000）,，使用证书模式手动确认会报错，原因不明
                        Thread.Sleep(2000);//模拟耗时 
                        Console.WriteLine(" [x] Done {0}", message);

                        // 发送消息确认信号（手动消息确认）
                        channel.BasicAck(deliveryTag: ea.DeliveryTag, multiple: false);
                    };
                    //7. 启动消费者
                    /*
                     autoAck:true；自动进行消息确认
                     autoAck:false；关闭自动消息确认，通过调用BasicAck方法手动进行消息确认
                     */
                    //channel.BasicConsume(queue: "test1", autoAck: true, consumer: consumer);
                    channel.BasicConsume(queue: "test1", autoAck: false, consumer: consumer);
                    Console.WriteLine(" Press [enter] to exit.");
                    Console.ReadLine();
                }
            }
        }
    }
}
```

### 消息持久化

消息确认确保了即使消费端异常，消息也不会丢失能够被重新分发处理。但是如果RabbitMQ服务端异常，消息依然会丢失。除非我们指定durable:true，否则当RabbitMQ退出或奔溃时，消息将依然会丢失。通过指定durable:true，并指定Persistent=true，来告知RabbitMQ将消息持久化。

```csharp
namespace RabbitMqDemo.Send
{
    class Program
    {
        static void Main(string[] args)
        {
            ServicePointManager.ServerCertificateValidationCallback += (sender, cert, chain, sslPolicyErrors) => true;
            ConnectionFactory factory = new ConnectionFactory()
            {
                UserName = "admin",
                Password = "admin",
                AutomaticRecoveryEnabled = true,
                Ssl = new SslOption()
                {
                    CertPath = @"E:\git\RabbitMqDemo\RabbitMqDemo.Send\server.pfx",
                    CertPassphrase = "123123",
                    AcceptablePolicyErrors = SslPolicyErrors.RemoteCertificateNameMismatch |
                                                SslPolicyErrors.RemoteCertificateChainErrors,
                    Enabled = true
                },
                //AuthMechanisms = new AuthMechanismFactory[] { new ExternalMechanismFactory() },
                RequestedHeartbeat = 60,
                Port = 5673,
                TopologyRecoveryEnabled = true
            };

            // 创建connection
            using (var connection = factory.CreateConnection(new string[1] { "192.168.0.115" }))
            {
                // 创建一个channel信道
                using (var channel = connection.CreateModel())
                {
                    //将消息标记为持久性 - 将IBasicProperties.SetPersistent设置为true
                    var properties = channel.CreateBasicProperties();
                    properties.Persistent = true;

                    // 申明队列 durable:是否持久化
                    var result = channel.QueueDeclare(queue: "test1", durable: true, exclusive: false, autoDelete: false, arguments: null);

                    for (int i = 0; i <21; i++)
                    {
                        //构建byte消息数据包
                        var body = Encoding.UTF8.GetBytes(i.ToString() + "test");
                        channel.BasicPublish(exchange: "", routingKey: "test1", basicProperties: properties, body: body);
                        Console.WriteLine("{0} 推送成功", i);
                        Thread.Sleep(2000);
                    }
                }
            }
            Console.Read();
        }
    }
}
```

需要注意的是，将消息设置为持久化并不能完全保证消息不丢失。虽然他告诉RabbitMQ将消息保存到磁盘上，但是在RabbitMQ接收到消息和将其保存到磁盘上这之间仍然有一个小的时间窗口。 RabbitMQ 可能只是将消息保存到了缓存中，并没有将其写入到磁盘上。持久化是不能够一定保证的，但是对于一个简单任务队列来说已经足够。如果需要消息队列持久化的强保证，可以使用[publisher confirms](https://www.rabbitmq.com/confirms.html)

### 公平分发

RabbitMQ的消息分发默认按照消费端的数量，按顺序循环分发。这样仅是确保了消费端被平均分发消息的数量，但却忽略了消费端的闲忙情况。这就可能出现某个消费端一直处理耗时任务处于阻塞状态，某个消费端一直处理一般任务处于空置状态，而只是它们分配的任务数量一样。

![2799767-d2bb1f2ac63fdb15.png](/img/2799767-d2bb1f2ac63fdb15.png)

但我们可以通过`channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);`
设置`prefetchCount : 1`来告知RabbitMQ，在未收到消费端的消息确认时，不再分发消息，也就确保了当消费端处于忙碌状态时，不再分配任务。

消费者Receive代码:

```csharp
// 申明队列
channel.QueueDeclare(queue: "test1", durable: true, exclusive: false, autoDelete: false, arguments: null);
//设置prefetchCount : 1来告知RabbitMQ，在未收到消费端的消息确认时，不再分发消息，也就确保了当消费端处于忙碌状态时
channel.BasicQos(prefetchSize: 0, prefetchCount: 1, global: false);
```

这时你需要注意的是如果所有的消费端都处于忙碌状态，你的队列可能会被塞满。你需要注意这一点，要么添加更多的消费端，要么采取其他策略。

## 几种 Exchange 模式

AMQP协议中的核心思想就是生产者和消费者隔离，生产者从不直接将消息发送给队列。生产者通常不知道是否一个消息会被发送到队列中，只是将消息发送到一个交换机。先由Exchange来接收，然后Exchange按照特定的策略转发到Queue进行存储。同理，消费者也是如此。Exchange 就类似于一个交换机，转发各个消息分发到相应的队列中。

RabbitMQ提供了Exchange，它类似于路由器的功能，它用于对消息进行路由，将消息发送到多个队列上。Exchange一方面从生产者接收消息，另一方面将消息推送到队列。但exchange必须知道如何处理接收到的消息，是将其附加到特定队列还是附加到多个队列，还是直接忽略。而这些规则由exchange type定义，exchange的原理如下图所示。

![2799767-0b4fba202e525745.png](/img/2799767-0b4fba202e525745.png)

### Exchange分类

RabbitMQ的Exchange（交换器）分为四类：

* ** direct**：默认；明确的路由规则：消费端绑定的队列名称必须和消息发布时指定的路由名称一致
* ** fanout**： 消息广播，将消息分发到exchange上绑定的所有队列上
* ** topic**：模式匹配的路由规则：支持通配符
* ** headers**：不常用，允许你匹配AMQP消息的header而非路由键，除此之外headers交换器和direct交换器完全一致，但性能却很差，几乎用不到

** 注意：** fanout、topic交换器是没有历史数据的，也就是说对于中途创建的队列，获取不到之前的消息。

#### 1，direct

路由机制如下图，即队列名称和消息发送时指定的路由完全匹配时，消息才会发送到指定队列上。

routingKey => direct exchange => queue

![2799767-6c78ab57fe06c6ce.png](/img/2799767-6c78ab57fe06c6ce.png)

生产者Send代码：

```csharp
ConnectionFactory factory = new ConnectionFactory()
{
    UserName = "admin",
    Password = "admin",
    AutomaticRecoveryEnabled = true,
    Ssl = new SslOption()
    {
        CertPath = @"E:\git\RabbitMqDemo\RabbitMqDemo.Send\server.pfx",
        CertPassphrase = "123123",
        AcceptablePolicyErrors = SslPolicyErrors.RemoteCertificateNameMismatch |
                                    SslPolicyErrors.RemoteCertificateChainErrors,
        Enabled = true
    },
    //AuthMechanisms = new AuthMechanismFactory[] { new ExternalMechanismFactory() },
    RequestedHeartbeat = 60,
    Port = 5673,
    TopologyRecoveryEnabled = true
};

// 创建connection 
using (var connection = factory.CreateConnection(new string[1] { "192.168.0.115" }))
{
    // 创建一个channel信道
    using (var channel = connection.CreateModel())
    {
        channel.ExchangeDeclare(exchange: "directEC", type: "direct");

        //将消息标记为持久性 - 将IBasicProperties.SetPersistent设置为true
        var properties = channel.CreateBasicProperties();
        properties.Persistent = true;

        for (int i = 0; i < 21; i++)
        {
            //构建byte消息数据包
            var body = Encoding.UTF8.GetBytes(i.ToString() + "test");
            channel.BasicPublish(exchange: "directEC", routingKey: "direct-key", basicProperties: null, body: body);
            Console.WriteLine("{0} 推送成功", i);
            //Thread.Sleep(2000);
        }
    }
}
```

消费者Receive代码：

```csharp
ConnectionFactory factory = new ConnectionFactory()
{
    UserName = "admin",
    Password = "admin",
    AutomaticRecoveryEnabled = true,
    Port = 5672,
    TopologyRecoveryEnabled = true
};

//  建立连接
using (var connection = factory.CreateConnection(new string[1] { "192.168.0.115" }))
{
    //  创建信道
    using (var channel = connection.CreateModel())
    {
        //申明direct类型exchange
        channel.ExchangeDeclare(exchange: "directEC", type: "direct");

        var queueName = channel.QueueDeclare().QueueName;
        channel.QueueBind(queue: queueName, exchange: "directEC", routingKey: "direct-key");

        var consumer = new EventingBasicConsumer(channel);

        //  绑定消息接收后的事件委托
        consumer.Received += (model, ea) =>
        {
            // var body = ea.Body;涉及到闭包，必须赋予变量
            var body = ea.Body;
            var message = Encoding.UTF8.GetString(body);
            Console.WriteLine(" [x] Received {0}", message);
            Console.WriteLine(" [x] Done1 {0}", message);
        };

        channel.BasicConsume(queue: queueName,
                             autoAck: true,
                             consumer: consumer);
        //channel.BasicConsume(queue: queueName, autoAck: true, consumer: consumer);
        Console.WriteLine(" Press [enter] to exit.");

        Console.ReadLine(); // 必须存在
    }
}
```

#### 2，fanout

发布/订阅模式

fanout的路由机制如下图，即发送到 fanout 类型exchange的消息都会分发到所有绑定该exchange的队列上去。

![2799767-3afd7b874221a9a2.png](/img/2799767-3afd7b874221a9a2.png)

生产者Send代码：

```csharp
// 创建connection 
using (var connection = factory.CreateConnection(new string[1] { "192.168.0.115" }))
{
    // 创建一个channel信道
    using (var channel = connection.CreateModel())
    {
        channel.ExchangeDeclare(exchange: "fanoutEC", type: "fanout");

        //将消息标记为持久性 - 将IBasicProperties.SetPersistent设置为true
        var properties = channel.CreateBasicProperties();
        properties.Persistent = true;

        for (int i = 0; i < 21; i++)
        {
            //构建byte消息数据包
            var body = Encoding.UTF8.GetBytes(i.ToString() + "test");

            //发布到指定exchange，fanout类型无需指定routingKey
            channel.BasicPublish(exchange: "fanoutEC", routingKey: "", basicProperties: properties, body: body);
            Console.WriteLine("{0} 推送成功", i);
            //Thread.Sleep(2000);
        }
    }
}
```

消费者Receive代码：

```csharp
//  建立连接
using (var connection = factory.CreateConnection(new string[1] { "192.168.0.115" }))
{
    //  创建信道
    using (var channel = connection.CreateModel())
    {
        //申明direct类型exchange
        channel.ExchangeDeclare(exchange: "fanoutEC", type: "fanout");

        var queueName = channel.QueueDeclare().QueueName;

        // 绑定队列到指定fanout类型exchange，无需指定路由键
        channel.QueueBind(queue: queueName, exchange: "fanoutEC", routingKey: "");

        var consumer = new EventingBasicConsumer(channel);

        //  绑定消息接收后的事件委托
        consumer.Received += (model, ea) =>
        {
            // var body = ea.Body;涉及到闭包，必须赋予变量
            var body = ea.Body;
            var message = Encoding.UTF8.GetString(body);
            Console.WriteLine(" [x] Received {0}", message);
            Console.WriteLine(" [x] Done1 {0}", message);
        };

        channel.BasicConsume(queue: queueName,
                             autoAck: true,
                             consumer: consumer);
        //channel.BasicConsume(queue: queueName, autoAck: true, consumer: consumer);
        Console.WriteLine(" Press [enter] to exit.");

        Console.ReadLine(); // 必须存在
    }
}
```

#### 3，topic

匹配订阅模式

topic是direct的升级版，是一种模式匹配的路由机制。它支持使用两种通配符来进行模式匹配：符号`#`和符号`*`。其中`*`匹配一个单词， `#`则表示匹配0个或多个单词，单词之间用`.`分割。如下图所示。

![2799767-3196a1c3880b3466.png](/img/2799767-3196a1c3880b3466.png)

生产者Send代码：

```csharp
// 生成随机队列名称
var queueName = channel.QueueDeclare().QueueName;
//使用topic exchange type，指定exchange名称
channel.ExchangeDeclare(exchange: "topicEC", type: "topic");
var message = "Hello Rabbit!";
var body = Encoding.UTF8.GetBytes(message);
//发布到topic类型exchange，必须指定routingKey
channel.BasicPublish(exchange: "topicEC", routingKey: "first.green.fast", basicProperties: null, body: body);
```

消费者Receive代码：

```csharp
//申明topic类型exchange
channel.ExchangeDeclare (exchange: "topicEC", type: "topic");
//申明随机队列名称
var queuename = channel.QueueDeclare ().QueueName;
//绑定队列到topic类型exchange，需指定路由键routingKey
channel.QueueBind (queue : queuename, exchange: "topicEC", routingKey: "#.*.fast");
```

### RPC

RPC是指远程过程调用，也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。

![python-six.png](/img/python-six.png)

![45366c44f775abfd0ac3b43bccc1abc3_hd.jpg](/img/45366c44f775abfd0ac3b43bccc1abc3_hd.jpg)

项目：[https://github.com/syxdevcode/RabbitMqDemo.git](https://github.com/syxdevcode/RabbitMqDemo.git)

参考官网：

[http://www.rabbitmq.com/tutorials/tutorial-six-dotnet.html](http://www.rabbitmq.com/tutorials/tutorial-six-dotnet.html)

参考：

[RabbitMQ Tutorials](http://www.rabbitmq.com/getstarted.html)

[RabbitMQ学习系列（一）: 介绍](http://www.cnblogs.com/zhangweizhong/p/5687457.html)

[RabbitMQ学习系列（四）: 几种Exchange 模式](http://www.cnblogs.com/zhangweizhong/p/5713874.html)

[RabbitMQ知多少](http://www.cnblogs.com/sheng-jie/p/7192690.html)

[深入解读RabbitMQ工作原理及简单使用](https://www.cnblogs.com/vipstone/p/9275256.html)

[RabbitMQ交换器Exchange介绍与实践](https://www.cnblogs.com/vipstone/p/9295625.html)