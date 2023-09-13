## 分布式
### Quorum、WARO

### paxos

### raft
* 领导人选举
* 日志复制
* 安全性约束

### zab
* 崩溃恢复模式
* 消息广播模式

### 负载均衡策略
* 轮询法
* 加权轮询法
* 随机法
* 加权随机法
* 源地址哈希法
* 最小连接数法

### 分布式系统设计目标
* 可拓展
* 高可用
* 无状态
* 可管理
* 高可靠

### 分布式事务解决方案
* 基于XA协议：两阶段提交和三阶段提交，需要数据库层面支持
* 基于事务补偿机制的：tcc，基于业务层面实现
* 基于事务消息：mq

### 二阶段和三阶段的问题和区别

### zookeeper

### 服务降级、服务熔断

### 高并发限流
* 计数器法
* 滑动窗口计数
* 漏桶算法
* 令牌桶算法

### 分布式缓存寻址算法
* hash法
* 一致性hash
* hash slot

## redis
#### 缓存穿透、缓存击穿、缓存雪崩

#### 缓存过期策略
* 定时过期
* 惰性过期
* 定期过期

#### 缓存淘汰算法
* FIFO
* LRU
* LFU

#### 如何保证数据库和缓存一致性
* 先更新数据库，再更新缓存
* 先删除缓存，再更新数据库
* 先更新数据库，再删除缓存
* 延时双删


#### 持久化机制
* RDB
    通过fork一个子进程，利用copy on write的技术来复制redis内存里的数据
* AOF

#### redis单线程为什么这么快
* 基于内存实现
* 高效的数据结构
* 核心单线程
* io多路复用

#### redis主从同步机制


#### redis高可用方案
* 主从
* 哨兵
* cluster
* redis sharding

#### redis事务实现

#### redis九大数据结构
* string
* list
* hash
* set
* sorted set
* bitmap
* geohash
* hyperloglog
* streams

#### redis分布式锁
* setnx + setex
* setnx(key, value, nx, px)

### kafka
#### kafka的架构
producer consumer broker(topic, partition) zookeeper

#### kafka高吞吐量的原因
* 顺序读写磁盘
* 零拷贝
* 分区分段+索引
* 批量压缩发送
* 使用pagecache机制间隔时间刷盘fsync

#### kafka的副本同步机制

#### kafka消息的高可靠方案

#### kafka的rebalance机制

### rocketmq
#### rocketmq的架构
producer consumer nameserver broker(topic, queue) 

#### rocketmq怎么实现事务消息

#### rocketmq如何实现顺序消息


#### rocketmq消息持久化机制

#### rocketmq如何保证消息不丢失
生产者
* 同步阻塞发送，失败重试，查询确认
* 异步发送监听回调
* ack，可能存储commitlog成功，存comsumerQueue失败，对消费者不见

broker：同步刷盘，集群同步复制


### 数据库
#### 如何分库分表
* 垂直：垂直分出来的结构是不一样的，比如把用户表和订单表分到两个库，把同一表中的两列分到两个表中
* 水平：水平分出来的结构还是一样的，只是内容不一样，比如用户信息根据地方进行分库或者分表
问题：
* 跨库事务
* 跨库关联
* 聚合函数
* 分布式id

#### 分布式id
* uuid：简单，性能好，没有顺序没有业务意义
* 雪花算法

### es
#### 