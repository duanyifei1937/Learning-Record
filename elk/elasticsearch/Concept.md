# Concept

## Document Index REST API

* **Index 索引**
    * Type 类型
    * **Document 文档**

* Node 节点
    * Shard 分片

### Document 文档
* 搜索的最小单位，被序列化为json格式存储在es中；
* 有自己的unique ID, 字段对应的数据类型；

### Index
* 文档的容器；
* Mapping: 定义文档字段类型
* Settings: 定义不同数据分布；

### REST API
* indices
    * create index:
        * put Movies
    * show all index:
        * _cat/indices

``` yaml
# some rest api:

```

## 集群/节点/分片/副本

### 节点
* 类型：
    * Master-eligible node: 参加选主，可成为master;
    * Master node: 可修改集群状态；(节点信息、索引信息、分片路由信息；)
    * Data node: 保存数据；
    * Coordinating node: 接受client请求，把请求分发到合适节点，结果汇聚；
    * Hot & Warm node

### 分片
* 主分片：
    * 一个分片是一个运行的lucene实例；
    * 􏵤􏱧􏵕􏴕􏵄􏵁􏵂􏵪􏱭􏵫􏵬􏲭􏰢􏲱􏵭􏴥􏵮􏵯􏴜􏲕􏰢􏵰􏳑􏰊􏰍􏰁􏰂􏴋􏰍􏰄􏵤􏱧􏵕􏴕􏵄􏵁􏵂􏵪􏱭􏵫􏵬􏲭􏰢􏲱􏵭􏴥􏵮􏵯􏴜􏲕􏰢􏵰􏳑􏰊􏰍􏰁􏰂􏴋􏰍􏰄􏵤􏱧􏵕􏴕􏵄􏵁􏵂􏵪􏱭􏵫􏵬􏲭􏰢􏲱􏵭􏴥􏵮􏵯􏴜􏲕􏰢􏵰􏳑􏰊􏰍􏰁􏰂􏴋􏰍􏰄􏵤􏱧􏵕􏴕􏵄􏵁􏵂􏵪􏱭􏵫􏵬􏲭􏰢􏲱􏵭􏴥􏵮􏵯􏴜􏲕􏰢􏵰主分片index创建是指定，不允许修改，除非reindex;
* 副本分片：
    * 动态调整；
    * 增加读取吞吐；
* 优化：
    * 默认一个节点可以存放所有分片数量，需要根据节点数量、shard数限制每个节点可以存放的shard数量；􏰊􏰍􏰁􏰂􏴋􏰍􏰄
    

