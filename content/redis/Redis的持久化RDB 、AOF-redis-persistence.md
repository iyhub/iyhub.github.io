---
title: Redis的持久化RDB 、AOF
date: 2021-11-25 18:05:34.157
updated: 2021-11-25 18:05:46.708
url: /archives/redis-persistence
categories: Redis
tags: [Redis  | 中间件]
---

# Redis的持久化
Redis是基于内存的，必须要考虑的一个问题就是：当redis服务挂了之后，之前的数据怎么重新加载到内存？那么必须要考虑一种存放历史数据的解决方案。
## RDB：指定的时间间隔对数据进行快照存储。
![](https://shanhai-blog.oss-cn-shanghai.aliyuncs.com/4fc2bb23-f621-4864-a593-39574136d3d5_20210719172717.png)


假如开始拍快照时间为10：00，整个快照落库持久化完成大概需要5分钟，即10：05此次快照拍完。而在10：04的时候b被改成了5，那么在rdb生成的结果里b是存的4还是5呢？是的，用大腿想也能想到肯定是存的4，但是如果让你来着设计这个RDB持久化怎么去实现呢?
- A方案:使用一个进程阻塞系统,系统拍照的时候我不准用户改动数据，等我快照拍完了你再改。此路不通。
- B方案:使用一个进程且不阻塞系统，继续让用户改数据，并同时将数据持久化落库，那就意味着一有数据改动就得重新拍照，那这个快照拍到什么时候能生成？此路不通。
- C方案:派（fork）一个子进程去把10：00当时那个点的数据a先落库（bgsave），主进程继续为用户服务，当用户改了数据b时，触发copyonwrite 将b的**原始引用**拷贝给子进程，主线程里的b引用新指向内存里的5。


![](https://shanhai-blog.oss-cn-shanghai.aliyuncs.com/e6b60b5b-4b58-42c8-af15-f62764694d49_20210719172748.png)

RDB`save`参数配置

```
save 900 1 # 每900s有1个key变化，时候生成RDB
save 300 10 # 每300s有10个key变化，时候生成RDB
save 60 10000 # 每60s有10000个key变化，时候生成RDB
```
- RDB的优点
  - 适用于灾难恢复，也可以根据需求恢复到不同版本的数据集.
  - 与AOF相比,在恢复大的数据集的时候，RDB方式会更快
  - 最大化redis的性能，存RBB主进程只管fork子进程。
- RDB的缺点
  - 可能会丢失数据
  - 虽然fork同步操作非常快，同步大数据量时，fork也会阻塞主进程

## Redis AOF(append only file)持久化
## 1. 开启AOF，AOF 特点

```
############################## APPEND ONLY MODE ###############################
appendonly yes # 默认为no,redis默认是rdb模式；
# aof 记录的文件名

appendfilename "appendonly.aof"

#每次写入仅追加日志后进行Fsync。缓慢但安全的
# appendfsync always
# Compromise（折中）方案，一秒追加一次。
appendfsync everysec 
# 由操作系统调度刷新输出缓冲区写入到aof文件
# appendfsync no
```



特点：
  - 记录的是数据操作流水，丢失数据少
  - 4.0版本之后支持开启RDB+AOF，默认关闭，可以通过配置项 aof-use-rdb-preamble 开启）

## 2. AOF 存在的问题：

随着日期增加 AOF 日志变得无限大，回复将会显得日志臃肿，表现为数据恢复耗时久。
重写之前已经记录下来的数据指令；

- 在 4.0 之前: 删除抵消的命令。like:incre key 1 & decre key 1，set key 1 & delete key

- 4.0 之后：支持开启：aof-use-rdb-preamble 将老的数据先生成 RD 再增量的通过 AOF 添加到日志。

## 3. appendonly.aof 日志格式

```
# set key2  testvalue
*3
$3
set
$4
key2
$9
testvalue

# 执行 mset key1 1 key2 2 key33 3
# aof日志如下：
*7 # 本批命令需要往下读7行非 $ 开始的命令
$4 #接着读取4个字节宽度，‘mset’长度为4，记为 $4
mset
$4 #接着读取4个字节宽度，‘key1’长度为4，记为 $4
key1
$1 #接着读取1个字节宽度，‘1’长度为1，记为 $1
1
$4
key2
$1
2
$5 #接着读取的字节宽度，‘$key33’长度为5，记为 $5
key33
$1
3
```

## 4. aof 日志什么时候追加？

redis 提供三种追加频率：

- no

  不要立刻刷，只有在操作系统需要刷的时候再刷 ，比较快。如果 redis 重启了拿到的数据将不是很新的数

- always

  每次写操作都立刻写入到 aof 文件。慢，但是最安全

- everysec

  每秒写一次 （默认）