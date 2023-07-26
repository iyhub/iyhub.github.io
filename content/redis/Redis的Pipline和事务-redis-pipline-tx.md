---
title: Redis的Pipline和事务
date: 2021-11-25 17:50:36.042
updated: 2021-11-25 17:52:03.434
url: /archives/redis-pipline-tx
categories: Redis
tags: [Redis  | 中间件]
---

# 1.Redis 管道（Pipeline）
  我们搬家的时候往往是大包小包。
  此时，有两种方案可以选择：
  - 完全靠一己之力，一趟一趟的搬。
  - 找个货运车，一趟就搬过去。

  而Pipeline就是这个这个货运车，把你的一批命令一把送过去，然后告诉你结果。目的是节省你的网络开销（建立连接 关闭连接 再开再关，对于计算机而已都是需要开辟资源去处理的）。
  ## 使用场景：
  1.批处理、维护缓存数据

  在电商的场景中，我们可能会把商品的一些信息放到缓存，比如说：库存数据，比如说显示的商品价格，而前篇文章提到了：
 [Redis中的List、Set、Hash实际使用场景](https://mp.weixin.qq.com/s/Xg2l--o41HaZDCC5kiDVTQ)
  我们用Redis Hash存放购物车数据，购物车的商品信息存在hash的一个filed：`info_productIdx`里。当我们的商品价格在后台被改掉的时候：购物车显示给用户的价格肯定也得是最新的。


# 2.Redis的发布发布订阅（Pub/Sub）

  发布订阅，顾名思义，一个消息发布之后，已经预约了当前主题的订阅者都能收到通知。
  ```SHELL
  127.0.0.1:6379> PUBLISH testChannel "hello sub"
  # 返回2为订阅者的个数
(integer) 2
  ```
  另外两个客户端订阅`testChannel`通道的消息，消息发布之后订阅者收到通知：
  ```
SUBSCRIBE testChannel
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "testChannel"
3) (integer) 1
1) "message"
2) "testChannel"
3) "hello sub"
1) "message"
  ```
订阅某个特定的前缀的通道：比如订阅下文中的所有秒杀活动开始的通知。
 ``` 
 SUBSCRIBE  activity*
 ```

## 使用场景:
1. 异步消息通知（like:活动预约提醒）
      我们在京东/天猫订阅的秒杀活动预约通知，其实我们往Redis技术层面来思考，Redis轻而易举就能实现了。
      假如活动ID为：666，活动key设置为：activity:666，活动未开始之前提供的那个【提醒我】，当按钮被点击之后，相当于用户订阅了通道key 为：`SUBSCRIBE activity:666`，而活动开始那一刻便是发布消息：`PUBLISH activity:666 "IPhone 12 activiy start "`，然后通过业务代码实现站内推送或者是短信的方式通知。

# 3.Redis事务
Redis事务相比MySQL，在Redis中，单条命令是原子性执行的，但事务不保证原子性，且没有回滚。事务中任意命令执行失败，其余的命令仍会被执行。
``` shell
# 开启事务：
MULTI
OK

INCR foo
QUEUED

INCR bar
QUEUED

# 提交事务
EXEC
# 提交结果，按提交命令顺序返回
1) (integer) 1
2) (integer) 1
```
-  Redis事务中的错误处理
   使用事务时可能会遇上以下两种错误：

    - 事务在执行 EXEC 之前，入队的命令可能会出错。比如说，命令可能会产生语法错误（参数数量错误，参数名错误，等等），或者其他更严重的错误，比如内存不足（如果服务器使用 maxmemory 设置了最大内存限制的话）。从 `Redis 2.6.5` 开始，服务器会对命令入队失败的情况进行记录，并在客户端调用 EXEC 命令时，`拒绝执行并自动放弃这个事务`。
   
   
    - 命令可能在 EXEC 调用之后失败。举个例子，事务中的命令可能处理了错误类型的键，比如将列表命令用在了字符串键上面，诸如此类。
    Redis不会进行特别处理： 即使事务中有某个/某些命令在执行时产生了错误， `事务中的其他命令仍然会继续执行`。

