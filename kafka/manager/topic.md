# Topic Manager

## 增
`bin/kafka-topics.sh --bootstrap-server broker_host:port --create --topic my_topic_name  --partitions 1 --replication-factor 1`

## 删
`bin/kafka-topics.sh --bootstrap-server broker_host:port --delete  --topic <topic_name>`
## 改
* monitor topic partitions:(only support add):
bin/kafka-topics.sh --bootstrap-server broker_host:port --alter --topic <topic_name> --partitions <新分区数>

* 修改主题级别参数:
bin/kafka-configs.sh --zookeeper zookeeper_host:port --entity-type topics --entity-name <topic_name> --alter --add-config max.message.bytes=10485760

* 变更副本数:
`kafka-reassign-partitions`
reference: https://duanyifei.cn/2018/07/20/kafka-scripts/#kafka-reassign-partitions-sh-1

* 修改主题限速:
bin/kafka-configs.sh --zookeeper zookeeper_host:port --alter --add-config 'leader.replication.throttled.rate=104857600,follower.replication.throttled.rate=104857600' --entity-type brokers --entity-name 0

bin/kafka-configs.sh --zookeeper zookeeper_host:port --alter --add-config 'leader.replication.throttled.replicas=*,follower.replication.throttled.replicas=*' --entity-type topics --entity-name test

* 主题分区迁移:

## 查
select topic:
`bin/kafka-topics.sh --bootstrap-server broker_host:port --list`

select single topic:
`bin/kafka-topics.sh --bootstrap-server broker_host:port --describe --topic <topic_name>`

## note
`--bootstrap-server -- 设置动态参数`
* how to manually reset offset
    * zk delete consumer, new consumer 从开始消费；
    * get offset, zk set offset values;
    * reference: https://community.cloudera.com/t5/Community-Articles/Manually-resetting-offset-for-a-Kafka-topic/ta-p/249368

## 特殊主题管理
* __consumer_offsets
kafka 0.11 < replication partion 问题

## topic error


