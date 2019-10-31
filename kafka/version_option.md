# 选型 & 部署
> kafka各个版本的对比

<!--ts-->
   * [选型](#选型)
      * [kafka 发行版](#kafka-发行版)
      * [kafka 监控工具](#kafka-监控工具)
      * [kafka 版本](#kafka-版本)

<!-- Added by: duanyifei, at: 2019年10月27日 星期日 22时40分25秒 CST -->

<!--te-->

## kafka 发行版
* apache kafka
* confluent kafka
* CDH kafka

## kafka 监控工具
* kafka egale
* kafka manager
* JMXTrans + InfluxDB + Grafana(~~)
* prometheus: jmx-export + kafka-export

## kafka 版本
* 0.7：基础消息队列；
* 0.8：副本
* 0.9：认证 & kafka connect;
* 0.10：start kafka streams;(consumer api 存在死锁问题)
* 0.11：幂等product api & 事务api & 消息格式重构(建议版本0.11.0.3)
* 1.0 & 2.0:  kafka streams功能  
**目前，线下ES 7.4其对应的kafka input client 2.1.0**
**服务端、客户端版本要一致**

## kafka压测
* 命令行脚本；

### kafka扩容broker
* ~~

## 部署
* OS选择(Linux recomment)
    * epoll IO模型 
    * Zero copy 
    * disk RAID
    * disk capacity  
        * retention day
        * message size
        * replica numbers
        * kafka compression
        * resource reserved
    * bandwidth：
        * 网卡最大值的70%(丢包)
        * resource reserved(2/3)             

## IO模型
* 阻塞式 I/O
* 非阻塞式 I/O
* I/O多路复用： select 
* 信号驱动 I/O: epoll
* 异步 I/O: windows IOCP 线程模型