---
title: "emqx共享订阅"
categories:
- MQTT
tags:
- MQTT
- emqx
date: 2023-07-15
---

# 什么是共享订阅
共享订阅是在多个订阅者之间实现负载均衡的订阅方式
```
                                                    [subscriber1] got msg1
             msg1, msg2, msg3                    /
[publisher]  ---------------->  "$share/g/topic"  -- [subscriber2] got msg2
                                                 \
                                                   [subscriber3] got msg3
```
共享订阅避免了多个订阅者重复消费消息的问题

# 共享订阅方式
共享订阅方式分为带群组的和不带群组的方式

**带群组的共享订阅**

以 $share/<group-name> 为前缀的共享订阅是带群组的共享订阅：

group-name 可以为任意字符串，属于同一个群组内部的订阅者将以负载均衡接收消息，但 EMQ X 会向不同群组广播消息。

例如，假设订阅者 s1，s2，s3 属于群组 g1，订阅者 s4，s5 属于群组 g2。那么当 EMQ X 向这个主题发布消息 msg1 的时候：

- EMQ X 会向两个群组 g1 和 g2 同时发送 msg1

- s1，s2，s3 中只有一个会收到 msg1

- s4，s5 中只有一个会收到 msg1

```
                                       [s1]
           msg1                      /
[emqx]  ------>  "$share/g1/topic"    - [s2] got msg1
         |                           \
         |                             [s3]
         | msg1
          ---->  "$share/g2/topic"   --  [s4]
                                     \
                                      [s5] got msg1
```

**不带群组的共享订阅**

以 $queue/ 为前缀的共享订阅是不带群组的共享订阅。它是 $share 订阅的一种特例，相当与所有订阅者都在一个订阅组里面
``` 
                                       [s1] got msg1
        msg1,msg2,msg3               /
[emqx]  --------------->  "$queue/topic" - [s2] got msg2
                                     \
                                       [s3] got msg3
```

# 参考文章
https://docs.emqx.com/zh/emqx/v4.0/advanced/shared-subscriptions.html