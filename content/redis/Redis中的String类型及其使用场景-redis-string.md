---
title: Redis中的String类型及其使用场景
date: 2021-11-25 17:30:01.879
updated: 2021-11-25 17:34:57.215
url: /archives/redis-string
categories: Redis
tags: [Redis  | 中间件]
---

## 1.先谈为什么Redis

既然已经有了 MySQL 数据库为什么还研发出 Redis 这种 key-value 内存数据库，或者为什么不直接存储在`.txt`/`.log`这种文件里？Redis 的出现为我们解决了什么问题？

- 解决磁盘的 IO 瓶颈
  - 存放在 txt/log、MySQL 的数据最终存放是在磁盘的，磁盘寻址耗时是`毫秒(ms)`级别的。
  - 磁盘存放的数据，一次查询能查询出来的数据大小受`带宽`（单位时间内能传递的文件大小）的影响。
- 而内存的寻址是`纳秒（ns）`级别的。相比磁盘寻址快了`十万`倍。
> 画外音1：为什么要MySQL使用索引？

> 画外音2：为什么Redis是单线程的？

## 2.String类型的数据存储

我们在使用 Redis 时候经常会用到对于某个 key 的 value 值进行自增，但是为什么 Redis 里面却没有 int 类型呢？

Redis 的存放的是字节数据，数据是类型是根据 encoding 编码获取的。
执行`set testkey 99`时 redis 会先判断这个 testkey 的 encoding 类型，判断出是个 int,下次 incr 的时候，就能感知到这个 testkey 对应的 value 是 int 类型。

```SHELL
# 查看testkey对应的value的编码类型，返回int
OBJECT encoding testkey
```

- 当我们存储的是 123 时，实际数据类型为 Int
- 当我们存储的是 abc 字符串时，实际数据类型是 embstr
- 当字符串很长（长度大于 44）时，实际数据类型为 raw
> 那么是否长度超过44为的数字：例如：99.....9，incr是不是就会报错？
  其实并不会，redis在incr之前也会判断一次encoding，判断是否能进行数值运算。

### 2.1 字符类型

- 常见操作：set、get、append、strlen
- 使用场景：
  - 存放手机动态验证码

### 2.2 数值类型

- 常见操作：incr、decr
- 使用场景：点赞数，加入购物车。

### 2.3 bitmap
- 位图常见操作：setbit、bitcount、bitpos、bitop
- 位图使用场景：
    - 统计系统上线后随机时间段内的用户登录天数。
    
      以用户ID+年份为key,value是一个位图(每一位代表第N天是否登录了，登录在位图中则记为1)；例如：获取用户ID为1006在2020年在`最后三天`登录了几次：
      ```shell
      bitcount 10062020 -3 -1
      ```
    - 某一年的某个时间段的活跃用户统计。
      以日期(yyyyMMdd)为key，每一个位代表一个用户。例如：在20210614，用户ID：1001登录系统，在20210615，用户ID：1002登录系统，在20210616，用户ID：1003登录系统；获取这三天内活跃用户数：
    ```
      setbit 20210614  1001  1
      setbit 20210615  1002  1
      setbit 20210616  1003  1
      
      # 三天内任何一个用户登录都计入到destkey
      bitop or destkey 20210614 20210615 20210616
      # 获取三天登录了的用户数
      bitcount destkey 0 -1
    ```
  > 小的优化点：能否借鉴HashMap的计算key的摆放位置：bitIndex = hash(userID)%(totalUserCount+N)，可以缩小位图value的长度，从而提高内存空间利用率？
  
  
  ### 下期预告：

	Redis中的List、Hash类型及其适用场景，敬请期待。