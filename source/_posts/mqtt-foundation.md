---
title: "MQTT"
categories:
- MQTT
tags:
- MQTT
date: 2023-07-15
---

## MQTT基本概念
**什么是MQTT**

MQTT（Message Queuing Telemetry Transport，消息队列遥测传输协议）是一种轻量级的、基于发布/订阅模式的通讯协议，专为物联网（IoT）设计。其主要特点包括轻量级、低延迟、支持多种质量服务（QoS）级别、高效的发布/订阅模型以及良好的网络适应性。(轻量级的发布订阅通讯协议，具有低延迟、多种qos（质量服务级别、良好的网络适应性的特点）)

**MQTT与其他协议（如HTTP）相比，有何主要区别？**

MQTT和HTTP在设计理念、使用场景和性能特点上有显著不同。MQTT基于发布/订阅模式，适用于带宽有限、延迟敏感和不稳定的网络环境，如物联网设备；而HTTP基于请求/响应模式，主要用于网页浏览、文件传输和API服务。MQTT协议头小，消息开销低，支持长连接和多种QoS级别，而HTTP协议头相对较大，适合传输大数据，如文件、图片等。

## MQTT协议细节

**MQTT协议中定义了哪几种消息质量（QoS）等级？请解释它们的含义。**

QoS 是消息的发送方（Sender）和接受方（Receiver）之间达成的一个协议

**MQTT协议定义了三种QoS等级**：
- QoS 0：至多一次（At Most Once），消息最多被传递一次，适用于可以容忍消息丢失的场景。(代表，Sender 发送的一条消息，Receiver 最多能收到一次，也就是说 Sender 尽力向 Receiver 发送消息，如果发送失败，也就算了)
- QoS 1：至少一次（At Least Once），确保消息至少被传递一次，但可能会重复传递，适用于不能容忍消息丢失但可以容忍重复的场景。(代表，Sender 发送的一条消息，Receiver 至少能收到一次，也就是说 Sender 向 Receiver 发送消息，如果发送失败，会继续重试，直到 Receiver 收到消息为止，但是因为重传的原因，Receiver 有可能会收到重复的消息)
- QoS 2：恰好一次（Exactly Once），确保消息精确地被传递一次，适用于需要高可靠性的场景。(代表，Sender 发送的一条消息，Receiver 确保能收到而且只收到一次，也就是说 Sender 尽力向 Receiver 发送消息，如果发送失败，会继续重试，直到 Receiver 收到消息为止，同时保证 Receiver 不会因为消息重传而收到重复的消息。)

**注意**：

QoS是发送方和接受方之间的协议，而不是发布者和订阅者之间的协议。换句话说，发布者发布了一条QoS1的消息，只能保证Broker能至少收到一次这个消息；而对于订阅者能否至少收到一次这个消息，还要取决于订阅者在订阅的时候和Broker协商的QoS等级。

**QoS降级**

当发布者和订阅者设置的qos不同时，MQTT代理（Broker）会选择两者当中qos等级最小的那一个发送消息给订阅者
```
Actual Subscribe QoS = MIN(Publish QoS, Subscribe QoS)
```
假设发布者qos设置成qos1，而订阅者qos设置成qos0，那么当消息发送给到Broker时Broker会按照订阅者的qos来发送消息给订阅者


**MQTT协议的发布/订阅流程**

发布/订阅流程包括发布者（Publisher）将消息发布到特定的主题（Topic），订阅者（Subscriber）通过订阅这些主题来接收消息。MQTT代理（Broker）负责处理消息的发布和订阅，将消息从发布者转发给所有订阅了相应主题的订阅者。

## MQTT与其他技术的比较
MQTT、RabbitMQ和Kafka都是流行的消息队列系统，但它们在设计目标、应用场景和性能方面有所不同。MQTT专为物联网设计，具有轻量级、低延迟和高效发布/订阅模型的特点；RabbitMQ是一个通用的消息队列系统，支持多种协议和灵活的路由机制；Kafka则是一个分布式流处理平台，适用于高吞吐量的数据处理场景。

## 参考
https://zhuanlan.zhihu.com/p/80203905