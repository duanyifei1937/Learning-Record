# 集群参数配置

## Broker
### 存储信息
* `log.dirs`: 数据存储多个路径 *
    * (不同磁盘挂载到不同路径使用：提高读写性能 & filaover(v1.1))
        * failover: (这是 Kafka 1.1 版本新引入的强大功能。要知道在以前，只要 Kafka Broker 使用的任何一块磁盘挂掉了，整个 Broker 进程都会关闭。但是自 1.1 开始，这种情况被修正了，坏掉的磁盘上的数据会自动地转移到其他正常的磁盘上，而且 Broker 还能正常工作。)(数据拷贝 or 冗余)
* `log.dir`

### ZK
* zk的作用：
    * 分布式协调管理、信息保存
* `zookeeper.connect`:
    * multip-kafka-cluster use single zk-cluster:
    zk1:2181,zk2:2181,zk3:2181/kafka1

### client-broker:
> 客户端连接broker

* `listeners`: 
* `advertised.listerers`:
* `host.name/port`: 

**配置统一使用hostname: zk中保存为hostname**

## Gloable Broker Data Keep
* `log.retention.{hours|minutes|ms}`:
* `log.retention.bytes`:
* `message.max.bytes`:
* `max.message.bytes`: broker接收的topic max message size

## Topic
* `auto.create.topics.enable`
* `unclean.leader.election.enable`：是否允许 Unclean Leader 选举
    * false(recommend): 不允许落后太多的replica竞选leader; (分区不可用)；
    * true: 丢消息；
* `auto.leader.rebalance.enable`：是否允许定期进行 Leader 选举(替换)
    * false(recommend): 
    * true: 定期partion leader重选举(更换Leader)(无性能收益)
* `retention.ms`
* `retention.bytes`
* `max.message.bytes`
    * 同时修改Broker: `replica.fetch.max.bytes` & Consumer: `fetch.message.max.bytes`

## Hot to set kafka topic variable(kafka scripts usage)
* create topic set

`bin/kafka-topics.sh --bootstrap-server localhost:9092 --create --topic transaction --partitions 1 --replication-factor 1 --config retention.ms=15552000000 --config max.message.bytes=5242880`


* modiry topic set(recomment)

`bin/kafka-configs.sh --zookeeper localhost:2181 --entity-type topics --entity-name transaction --alter --add-config max.message.bytes=10485760`


## JVM
* [offical recomment config}() or 6G
* GC Options:
    * CMS
        * `-XX:+UseCurrentMarkSweepGC`
    * 吞吐量
        * `-XX:+UseParallelGC`
    * G1(default)

* `KAFKA_HEAP_OPTS`:
* `KAFKA_JVM_PERFORMANCE_OPTS`:

设置环境变量？ or  修改配置文件？

``` yaml
$> export KAFKA_HEAP_OPTS=--Xms6g  --Xmx6g
$> export KAFKA_JVM_PERFORMANCE_OPTS= -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true
$> bin/kafka-server-start.sh config/server.properties
```

## System
* 文件描述符
* 文件系统：xfs (https://www.confluent.io/kafka-summit-sf18/kafka-on-zfs)
* swappiness: 1 or off
* commit time(flush 落盘时间)
    * default 5s --> > 5s(换取更大性能，用多副本冗余保证)
    * **(producer send message to broker how to success?)**
        * (producer acks=1 丢数据)

### kafka message write to disk steps:
* kafka --> system page cache --> LRU --> disk