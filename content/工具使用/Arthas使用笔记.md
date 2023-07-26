---
title: Arthas 使用笔记
date: 2022-06-13 23:01:29.366
updated: 2022-06-13 23:04:56.858
url: /archives/arthas
categories: 日常开发
tags: ["tool","arthas"]
---

## Arthas 使用笔记

### 1.查找某个类文件 DemoController,热加载class

```shell
sc com.arthas.controller.DemoController

# 在IDEA中修改,之后,编译
mc /Users/develop/source/arthas-demo/src/main/java/com/arthas/controller/DemoController.java


# 热加载
redefine /Users/develop/source/zhuyong/arthas-demo/com/arthas/controller/DemoController.class

```

### 2.获取线程状态

```shell
# 获取当前最忙碌的前三个线程
thread -n 3
# 显示指定线程的运行堆栈
thread 线程id

# 找出当前阻塞其他线程的线程
# 有时候我们发现应用卡住了， 通常是由于某个线程拿住了某个锁， 并且其他线程都在等待这把锁造成的。
thread -b
# 获取 WAITING 状态的线程
thread --state WAITING
thread --state TIMED_WAITING

# 找出当前阻塞其他线程的线程
thread -b
```

### 3. 获取 当前垃圾收集器,线程数量(jvm 命令)

```shell
jvm

---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 GARBAGE-COLLECTORS
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 PS Scavenge                                    name : PS Scavenge
 [count/time (ms)]                              collectionCount : 4
                                                collectionTime : 56

 PS MarkSweep                                   name : PS MarkSweep
 [count/time (ms)]                              collectionCount : 2
                                                collectionTime : 78

---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 THREAD
---------------------------------------------------------------------------------------------------------------------------------------------------------------------
 COUNT                                          34 # JVM当前活跃的线程数
 DAEMON-COUNT                                   30 # JVM当前活跃的守护线程数
 PEAK-COUNT                                     36 # 从JVM启动开始曾经活着的最大线程数
 STARTED-COUNT                                  51 # 从JVM启动开始总共启动过的线程次数
 DEADLOCK-COUNT                                 0  # JVM当前死锁的线程数
```

### 4.查询方法调用链路,以及方法耗时:

命令: trace 

eg: 

```shell
# trace com.letus.sttrp.service.impl.ReportLogServiceImpl getReportLogList
[arthas@35897]$ trace -n 3 com.xx.ReportLogServiceImpl getReportLogList
Press Q or Ctrl+C to abort.
Affect(class count: 2 , method count: 2) cost in 360 ms, listenerId: 1
`---ts=2021-02-05 16:22:37;thread_name=http-nio-8088-exec-12;id=c5;is_daemon=true;priority=5;TCCL=org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader@431e9b93
    `---[24.287345ms] com.xx.service.impl.ReportLogServiceImpl$$EnhancerBySpringCGLIB$$1:getReportLogList()
        `---[23.160698ms] org.springframework.cglib.proxy.MethodInterceptor:intercept()
            `---[21.818245ms] com.xx.service.impl.ReportLogServiceImpl:getReportLogList()
                +---[0.023195ms] com.xx.dto.report.response.ReportLogListResponse:<init>() #122
                +---[0.059662ms] com.xx.common.utils.UserSession:getCurrUserId() #123
                +---[0.022441ms] com.xx.common.utils.UserSession:getCurrUserId() #124
                +---[0.013002ms] com.xx.dto.report.request.ReportLogListRequest:setUserId() #124
                +---[21.402332ms] com.xx.service.impl.ReportLogServiceImpl$1:<init>() #126
                +---[0.017436ms] com.xx.utils.GeneratePageUtils:getResultList() #142
                `---[0.011632ms] com.xx.dto.report.response.ReportLogListResponse:setReportLogDtos() #142
```

### 据调用耗时过滤

[-n N] trace N 次

```shell
trace -n 3 com.xx.impl.ReportLogServiceImpl getReportLogList  '#cost > 10'
```

>  只会展示耗时大于10ms的调用路径，有助于在排查问题的时候，只关注异常情况

### 5.更专业的耗时分析

```shell
tt -t -n 3 com.xx.service.impl.ReportLogServiceImpl getReportLogList

tt -t -n 3 com.xx.service.impl.WarehouseInventoryServiceImpl warehouseAccountInventoryList

tt -t -n 3 com.xx.controller.WarehouseInventoryController warehouseAccountInventoryList

trace com.lxx.service.impl.WarehouseInventoryServiceImpl warehouseAccountInventoryList
```

### heapdump



