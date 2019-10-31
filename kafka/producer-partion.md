# 客户端实践及原理解析
> producer、consumer、broker结合使用原理

## Producer --> Partiton
### why use partition?
* 负载均衡、读写增加性能；
* 灵活扩缩容；

### 分区策略(选择策略)
* **RR**: (未定义key default RR)轮询策略有非常优秀的负载均衡表现，它总是能保证消息最大限度地被平均分配到所有分区上，故默认情况下它是最合理的分区策略，也是我们最常用的分区策略之一。
* **Randomness**: 随机、(message % partition)趋于
* **Key-ordering**: 给message定义key, 并保证相同key进入同一个partition;
* **地理位置**: 

## Producer Compression
### how to compression
* v1: (<0.11.0) 多条message压缩
* v2: (>=0.11.0) message set 压缩 (避免每条消息CRC)

### when to compression
**Producer 端压缩、Broker 端保持、Consumer 端解压缩**
* 两种特殊情况除producer外，broker也会压缩：
    * Broker 端指定了和 Producer 端不同的压缩算法
    * Broker 端发生了消息格式转换
        * (new/old client version && 丧失zero copy)

### when to discompression
**consumer + broker**
Broker: 每个压缩过的消息集合在 Broker 端写入时都要发生解压缩操作，目的就是为了对消息执行各种验证

### Compression Algorithm PK
![](media/15725325772030.jpg)

### Best Practice
权衡cpu & bandwidth资源, 是否使用压缩

#### issue
* [避免每条消息解压校验](https://issues.apache.org/jira/browse/KAFKA-8106)