---
title: 基于Canal中间件同步MySQL数据到Elasticsearch（V7.6.2）
date: 2021-11-25 17:59:44.091
updated: 2021-11-25 17:59:52.886
url: /archives/canal-mysql-es
categories: 大数据
tags: ["中间件"]
---

## 基于canal中间件同步MySQL数据到Elasticsearch（V7.6.2）
- 本文软件版本（不提版本都是耍流氓）
> OS: MacOS<br/>
> mysql(mac server): 8.0<br/>
> canal: 1.1.5发布版<br/>
> elasticsearch: 7.6.2<br/>
> kibana: 7.6.2<br/>
### 1.遇到的一些坑：
1. adapter 订阅不到 mysql 的日志
    - mysql是否开启了binlog
    - canal.adapter-1.1.5/lib 下默认是没有5.x的jar包
    - 阿里云的RDS我实操时候是通过的，但是睡一觉起来，数据又订阅不到了，就放弃了
2. <font color='red'>启动adapter时报：com.alibaba.druid.pool.DruidDataSource cannot be cast to com.alibaba.druid.pool.DruidDataSource</font>
    - 下载v1.1.5-alpha-2 版本下的client-adapter.es7x-1.1.5-SNAPSHOT-jar-with-dependencies.jar 替换原client-adapter.es7x-1.1.5-jar-with-dependencies.jar
    - 不要信改ES配置 hosts: 127.0.0.1:9200 -> hosts: http://127.0.0.1:9200 改了会提示你 hostName not found!

### 2.撸起袖子开始干：
- 开启mysql binlog
```shell
vi /etc/my.cnf
# 新增如下配置
log-bin = mysql-bin #开启binlog
binlog-format = ROW #选择row模式
server_id = 1 #配置mysql replication需要定义，不能和canal的slaveId重复
```
- 下载canal
下载canal.adapter-1.1.5、canal.deployer-1.1.5<br/>
（推荐一个油猴加速下载插件：
<a href="chrome-extension://dhdgffkkebhmkfjojejmpbldmpobfkfo/options.html#nav=31506985-d326-4320-bd42-998eb5a8b957+editor">Github 增强 - 高速下载）</a>
<a>https://github.com/alibaba/canal/releases/tag/canal-1.1.5</a>
- 配置canal服务端订阅mysql binlog
 >canal.deployer-1.1.5/conf/canal.properties 配置基本不用动<br/>
 >
 >修改：/canal-1.1.5/canal.deployer-1.1.5/conf/example/instance.properties
 ```
canal.instance.master.address=127.0.0.1:3306 #数据库
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

canal.instance.dbUsername=root #订阅的数据库账号
canal.instance.dbPassword=root #订阅的数据库密码

# table regex
canal.instance.filter.regex=.*\\..* #表筛选
# table black regex
canal.instance.filter.black.regex=mysql\\.slave_.*
 ```
- 配置canal-adapter客户端同步给ES的相关配置
canal-1.1.5/canal.adapter-1.1.5/conf/application.yml
```
  canalAdapters:
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger # 日志的
      - name: es7 #ES 相关
        hosts: 127.0.0.1:9200 # 不用填http://
        properties:
          mode: rest #  9200填写rest，transport：9300用
          # security.auth: test:123456 #  only used for rest mode
          cluster.name: elasticsearch
```

- 配置canal-adapter客户端ES和MySQL字段的映射
canal-1.1.5/canal.adapter-1.1.5/conf/es7/demo.yml
```
dataSourceKey: defaultDS
destination: example
groupId: g1 #
esMapping:
  _index: demo_index
  _id: id #这个字段和sql语句的查询别名一致
#  upsert: true
#  pk: id
  sql: "select id as id,name from demo"
#  objFields:
#    _labels: array:;
  commitBatch: 3000
```
- canal.deployer完整配置
canal-1.1.5/canal.deployer-1.1.5/conf/example/instance.properties
```
# enable gtid use true/false
canal.instance.gtidon=false

# position info
canal.instance.master.address=127.0.0.1:3306
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true

# username/password
canal.instance.dbUsername=root
canal.instance.dbPassword=root
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
canal.instance.filter.regex=.*\\..*
# table black regex
canal.instance.filter.black.regex=mysql\\.slave_.*

# mq config
canal.mq.topic=example
canal.mq.partition=0
#################################################

```
- canal-adapter服务端完整配置
canal-1.1.5/canal.adapter-1.1.5/conf/application.yml
```
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  mode: tcp #tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: 0
  timeout:
  accessKey:
  secretKey:
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 127.0.0.1:11111
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:
    # kafka consumer
    kafka.bootstrap.servers: 127.0.0.1:9092
    kafka.enable.auto.commit: false
    kafka.auto.commit.interval.ms: 1000
    kafka.auto.offset.reset: latest
    kafka.request.timeout.ms: 40000
    kafka.session.timeout.ms: 30000
    kafka.isolation.level: read_committed
    kafka.max.poll.records: 1000
    # rocketMQ consumer
    rocketmq.namespace:
    rocketmq.namesrv.addr: 127.0.0.1:9876
    rocketmq.batch.size: 1000
    rocketmq.enable.message.trace: false
    rocketmq.customized.trace.topic:
    rocketmq.access.channel:
    rocketmq.subscribe.filter:
    # rabbitMQ consumer
    rabbitmq.host:
    rabbitmq.virtual.host:
    rabbitmq.username:
    rabbitmq.password:
    rabbitmq.resource.ownerId:

  srcDataSources:
   defaultDS:
     url: jdbc:mysql://127.0.0.1:3306/数据库名?useUnicode=true
     username: root
     password: root
  canalAdapters:
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
      - name: es7
        hosts: 127.0.0.1:9200 # 127.0.0.1:9200 for rest mode
        properties:
          mode: rest # or rest
          # security.auth: test:123456 #  only used for rest mode
          cluster.name: elasticsearch
```


**<font color='red'>Last but not least : </font>再遇到异常那就上git 搜你的对应异常Issue,你踩得坑别人一定都踩过了！**