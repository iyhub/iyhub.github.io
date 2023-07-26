---
title: Redis 安装教程（单机、伪集群）
date: 2021-11-25 18:08:43.352
updated: 2021-11-25 18:08:43.352
url: /archives/redis-install
categories: Redis
tags: [Redis] 
---

# Redis 单机安装教程
```
# 安装wget工具
yum install wget
# 安装gcc,c语言编译工具
yum install gcc
# 创建soft,便于后期管理软件
mkdir soft
 
wget http://download.redis.io/releases/redis-5.0.5.tar.gz
 
tar xf redis-5.0.5.tar.gz
# 重命名一下
mv redis-5.0.5 redis
 
cd redis
 
#可以读读readme.md,其实能读取到很多信息
 
#linux编译redis源码
make
 
# 如果没有安装gcc,但是执行了make,构建了一次,需要清除一下编译环境
make disclean
 
make
 
cd src # 查看执行程序是否生成
 
#将redis 装在指定的目录 : /opt/redis
make install PREFIX=/opt/redis
 
# 将redis配置成全局可执行服务
vi /etc/profile
 
export REDIS_HOME=/opt/redis
export PATH=$PATH:$REDIS_HOME/bin
# 刷新 profile文件,使其生效
source /etc/profile
# 进入到你的redis文件夹的下的utils下
cd /soft/redis/utils
# 准备配置你的Redis所在的端口
 
./install_server.sh
# 设置你的端口
6381
# 指定配置文件
/opt/redis/6381.conf
# 指定log位置
/opt/redis/log/redis_6381.log
# 指定数据存放位置
/opt/redis/data/6381
# 确认你的server可执行文件放置位置,因为我们在profile配置过,这里能自动识别
/opt/redis/bin/redis-server
 
# 一个物理机可以一个应用有多个进程 开不同的port,相当于登录两个QQ呗
service redis_[port] start/stop/status 
```
# Redis主从架构，并设置哨兵实现主的HA

```
# 配置/opt/redis/6379.conf,启动6379 为master节点,配置文件我并没有修改 
# 按理来说redis默认的启动是单节点
daemonize yes #设置redis 为后台运行
# replicaof <masterip> <masterport> #当前任然是注释状态,没有放开
 
 
 
# 配置 6380 为第一台从节点,配置文件我放在了 /opt/redis/6380.conf
daemonize yes
 
replicaof 127.0.0.1 6379 #告诉当前从机,master 节点的ip和端口
 
 
# 同理:配置 6382 为第二台从节点,配置文件我放在了 /opt/redis/6382.conf
daemonize yes
 
replicaof 127.0.0.1 6379 #告诉当前从机,master 节点的ip和端口
 
 
#/opt/redis/26379.conf设置第一台哨兵的IP
port 26379
 
sentinel monitor mymaster 127.0.0.1 6379 2

 
 
 
 
#/opt/redis/26380.conf设置第二台哨兵的IP
port 26380

sentinel monitor mymaster 127.0.0.1 6379 2

 
 
 
#/opt/redis/26382.conf设置第三台哨兵的IP
port 26382
 
sentinel monitor mymaster 127.0.0.1 6379 2

 
 
 
 
#启动 对应的主从机器
redis-server /opt/redis/6379.conf
redis-server /opt/redis/6380.conf
redis-server /opt/redis/6382.conf
 
 
#启动 对应的哨兵机器
redis-server /opt/redis/26379.conf
redis-server /opt/redis/26380.conf
redis-server /opt/redis/26382.conf
 
# 查看一下进程
ps -ef | grep redis
 
 
root       7323      1  0 11:46 ?        00:00:06 redis-server 127.0.0.1:6379
root       7341      1  0 11:52 ?        00:00:05 redis-server 127.0.0.1:6380
root       5667      1  0 10:46 ?        00:00:17 redis-server 127.0.0.1:6382
root       7351      1  0 11:54 ?        00:00:11 redis-server *:26379 [sentinel]
root       7271      1  0 11:37 ?        00:00:13 redis-server *:26380 [sentinel]
root       7276      1  0 11:37 ?        00:00:14 redis-server *:26382 [sentinel]
 
```
