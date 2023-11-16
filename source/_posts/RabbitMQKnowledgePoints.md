---
title: RabbitMQ 知识点
tags:
  - 消息队列
categories:
  - 编程
date: 2019-12-19 14:02:39
---

> https://mp.weixin.qq.com/s/r8k1TN46Pk61qTjZ8ljsEQ

<!--more-->

&emsp;&emsp;目前，主流的消息中间件主要有：ActiveMQ、Kafka、RabbitMQ、RocketMQ 等等......而我们今天的主角是：RabbitMQ。RabbitMQ 是开源基于 erlang 语言开发，具有高可用高并发的优点，适合集群消息代理和队列服务器，它是基于 AMQP 协议来实现的。AMQP 的主要特征是面向消息、队列、路由（包括点对点和发布/订阅）、可靠性、安全。RabbitMQ 支持多种语言，有消息确认机制和持久化机制，保证数据不丢失的前提做到可靠性、可用性。

# 消息与消息队列
&emsp;&emsp;消息（Message）是指应用于应用之间传送的数据，消息的类型包括文本字符串、JSON、XML、内嵌对象等等...  
&emsp;&emsp;所谓 **消息中间件 / 消息队列**（Message Queue Middleware，简称 MQ）是利用高效可靠的消息传递机制进行数据交流，同时可以基于数据通信来进行分布式系统的继承。消息中间件一般有两种传递模式：**点对点（Point-to-Point）模式**和**发布/订阅（Pub/Sub）模式**。点对点模式是基于队列的，消息生产者发送消息到队列，消息消费者从队列中接收消息，队列的存在使得消息的异步传输成为了可能。发布订阅模式定义了如何向一个内容节点发布和订阅内容，这个内容节点叫 topic，这种模式可以满足消费者发布一个消息，多个消费者同时消费同一信息的需求。
![性能对比](1.webp)

## 什么是 AMQP 协议？
&emsp;&emsp;AMQP 的全称：Advanced Message Queuing Protocol（高级消息队列协议），它是消息队列的一个规范，其中定义了很多核心的概念，AMQP 与 JMS（Java Message Service）Java 平台的专业技术规范类似，同样提供了很多面向中间件的 API，用于两个应用程序之间，或者分布式系统之间的发送消息，进行异步通信。

## AMQP 的协议模型
&emsp;&emsp;**解释**：Producer（生产者）将信息投递到 Server 端的 RabbitMQ 的 Exchange 中（过程：message -> server -> virtual host -> RabbitMQ -> Exchange），Consumer（消费者）只需要订阅消息队列 Message Queue，每当有消息投递到队列 queue 中时，都会通知消费者进行消费。
* 生产者只需要将消息投递到 Exchange 交换机中，不需要关注消息被投递到哪个队列。
* 消费者只需要监听队列来消费消息，不需要关注消息来自于哪个 Exchange。
* Exchange 和 Message Queue 存在着绑定的关系，一个 Exchage 可以绑定多个消息队列。

![ ](2.webp)

## AMQP 核心概念
* **Server**：又称 Broker，接受客户端的连接，实现 AMQP 实体服务；
* **Connection**：连接，应用于程序与 Broker 的网络连接；
* **Channel**：网络通道，几乎所有的操作都是在 channel 中进行的，channel 是进行消息读写的通道，客户可以建立多个 channel，每个 channel 代表一个会话任务；
* **Message**：消息，服务器与应用程序之间传送的数据，由 Properties 和 Body 组成，Properties 可以对消息进行修饰，比如消息的优先级，延迟等高级特性，Body 是消息体内容；
* **Virtual host**：虚拟地址，用于进行逻辑隔离，最上层的消息路由，一个 Virtrual host 里面可以有若干个 Exchange 和 Queue，同一个 Virtrual host 里面不能有相同名字的 Exchange 或 Queue；
* **Exchange**：交换机，接收消息，根据路由键转发消息到绑定的队列；
* **Routingkey**：生产者架构消息发给交换器的时候，会指定一个 RoutingKey，用来置顶这个消息的路由规则，通过 RoutingKey 来决定消息流向哪里；
* **Binding**：绑定，RabbitMQ 中通过绑定将交换器跟队列关联起来，在绑定的时候会指定一个绑定键（BindingKey），这样 RabbitMQ 就知道如何正确地将消息路由到对应的队列中去了，也就是生产者将信息发送给交换器时，需要一个 RoutingKey，当 RoutingKey 与 BindingKey 完全匹配时，消息会被路由到对应的队列中去；
* **Queue**：也叫 Message Queue，消息队列，保存消息并将他们转发给消费者；

&emsp;&emsp;注意：对于初学者，交换器、路由键、绑定这几个概念理解起来比较晦涩，这里做个比喻：交换器相当于投递包裹的邮箱，RoutingKey 相当于填写在包裹上的地址，BindingKey 相当于包裹的目的地，当填写在包裹上的地址和实际想要投递的地址相匹配，那么这个包裹就会被投递到目的地，最后这个目的地的主人 "队列" 就可以保留这个包裹，如果对应的地址不匹配，也就是 RoutingKey 和 BindingKey 不匹配，邮递员就不能正确地投递到目的地，包裹可能会回退给寄件人，也可能被丢弃。  

## AMQP 消息路由
&emsp;&emsp;AMQP 中消息的路由过程和 JMS 存在一些差别。**AMQP** 中增加了 Exchange 和 Binding 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到哪个队列。
![ ](3.webp)

### Exchange 类型
&emsp;&emsp;RabbitMQ 常用的交换器类型主要有四种：direct、fanout、topic、headers ，Exchange 分发消息时根据类型的不同分发策略有区别，headers 匹配 AMQP 消息的 header 而不是路由键，此外 headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了，这里仅仅对其他三种进行展开说明：  

#### 1. direct
![ ](4.webp)

&emsp;&emsp;direct 类型的交换器理由规则需要遵循严格的完全匹配规则，他会把消息路由到那些 BindingKey 和 RoutingKey 完全匹配的队里中，如果消息中的路由键（RoutingKey）和 Binding 中的 BindingKey 一致， 交换器就将消息发到对应的队列中。比如：
![ ](5.webp)

* 在发送消息的时候设置路由键为 “info” 或者 “debug”，消息只会路由到 Queue2，如果以其他路由键发送消息，则消息不会路由到这两个队里中，这就是路由键和 Binding key 的完全匹配。
  
&emsp;&emsp;direct 类型模式的特点：
1. 不需要将 Exchange 进行任何绑定（binding）操作
2. 消息传递时需要一个 “RoutingKey”，可以简单的理解为要发送到的队列名字。
3. 如果 vhost 中不存在 RoutingKey 中指定的队列名，则该消息会被抛弃。

#### 2. fanout
![ ](6.webp)

&emsp;&emsp;**它会将所有发送到该交换器的消息路由到所有与该交换器绑定的队列中。**fanout 交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。就像子网广播，每台子网内的主机都获得了一份复制的消息。fanout 类型转发消息是最快的。  

&emsp;&emsp;这种模式的特点：
1. 不需要 RoutingKey，我们将路由键设置为空即可。
2. 需要提前将 Exchange 和 Queue 进行绑定，一个 Exchange 可以绑定多个 Queue，一个 Queue 可以同时与多个 Exchange 进行绑定（"多对多关系"）。
3. 如果接受到消息的 Exchange 没有与任何 Queue 绑定，则消息会被抛弃。

#### 3. topic
![ ](7.webp)

&emsp;&emsp;direct 类型的交换器理由规则需要遵循严格的完全匹配规则，这种严格的匹配方式有时候不能满足实际的业务需求，topic 就是在这种规则上进行了扩展，topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，但是这里的匹配规则有所不同，它的约定如下：
* RoutingKey 为一个点号 "." 分隔的字符串（被点号 "." 号分隔开的一段独立的字符串称为单词），比如："com.rabbit.client"、"java.util.Map";
* BindingKey 和 RoutingKey 为一个点号 "." 分隔的字符串；
* BindingKey 中可以存在两种特殊的字符串 "\*" 和 "\#"，用于做模糊匹配，其中 "\*" 用于匹配一个单词， "\#" 用于匹配多规则单词；
  * "com.#" 可以匹配到 com.rabbitmq.aaa
  * "com.*" 可以匹配到 com.rabbitmq

&emsp;&emsp;**举个实例**：
![ ](8.webp)
* 路由键为 "com.rabbitmq.client" 的消息会同时路由到 Queue1 和 Queue2
* 路由键为 "com.hidden.client" 的消息只会路由到 Queue2
* 路由键为 "com.hidden.data" 的消息只会路由到 Queue2
* 路由键为 "java.rabbitmq.data" 的消息只会路由到 Queue1
* 路由键为 "java.util.map" 的消息将会被丢弃或者返回给生产者，因为它没有匹配任何的路由键。

## RabbitMQ 特点
&emsp;&emsp;RabbitMQ 最初起源于金融系统，用于在分布式系统中存储转发消息，在易用性、扩展性、高可用性等方面表现不俗。具体特点包括：
1. 可靠性  
RabbitMQ 使用一些机制来保证可靠性，如持久化、传输确认、发布确认。
2. 灵活的路由（Flexible Routing）  
在消息进入队列之前，通过 Exchange 来路由消息。对于典型的路由功能，RabbitMQ 已经提供了一些内置的 Exchange 来实现。针对更复杂的路由功能，可以将多个 Exchange 绑定在一起，也可以通过插件机制实现自己的 Exchange。
3. 消息集群（Clustering）  
多个 RabbitMQ 服务器可以组成一个集群，形成一个逻辑 Broker 。
4. 高可用  
队列可以在集群中的机器上进行镜像，使得在部分节点出问题的情况下队列仍然可用。
5. 多种协议（Multi-protocol）  
RabbitMQ 支持多种消息队列协议，比如 STOMP、MQTT 等等。
6. 多语言客户端  
RabbitMQ 几乎支持所有常用语言，比如 Java、.NET、Ruby 等等。
7. 管理界面  
RabbitMQ 提供了一个易用的用户界面，使得用户可以监控和管理消息 Broker 的许多方面。
8. 跟踪机制  
如果消息异常，RabbitMQ 提供了消息跟踪机制，使用者可以找出发生了什么。
9. 插件机制  
RabbitMQ 提供了许多插件，来从多方面进行扩展，也可以编写自己的插件。

# RabbitMQ 中的概念模型
## 消息模型
&emsp;&emsp;所有 MQ 产品从模型抽象上来说都是一样的过程：  
&emsp;&emsp;消费者（consumer）订阅某个队列。生产者（producer）创建消息，然后发布到队列（queue）中，最后将消息发送到监听的消费者。
![消息流](9.webp)

## RabbitMQ 基本概念
&emsp;&emsp;上面只是最简单抽象的描述，具体到 RabbitMQ 则有更详细的概念需要解释。上面介绍过 RabbitMQ 是 AMQP 协议的一个开源实现，所以其内部实际上也是 AMQP 中的基本概念：
![RabbitMQ 内部结构](10.webp)

## RabbitMQ 的作用和使用场景
![ ](11.webp)

## RabbitMQ 的核心组件
![ ](12.webp)
