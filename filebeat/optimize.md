# filebeat 优化

## 存在的问题
日志收割延时


## 参考

[How to Tune Elastic Beats Performance: A Practical Example with Batch Size, Worker Count, and More
](https://www.elastic.co/blog/how-to-tune-elastic-beats-performance-a-practical-example-with-batch-size-worker-count-and-more)这篇文章给出了一些可优化的点，做如下总结：

### filebeat --> elasticsearch
只给出了filebeat direct to es, 如果增加logstash, 参考Wait, why did you mention Logstash? This is a Beats post.

### 测试方式
固定7.5w条日志，使用filebeat收割，查看beat monitor, 关注beat `evetn rate` & `Throughput` & 收割完成时间, 实现rate最大化；
(测试为什么不使用实时写入？ 压测生成log, buffer延时，并且可能造成IO瓶颈)
注意bandwidth limit;

### 优化内容
* batch size & worker
**根据场景使用调整，测试环境为单es(1g), 单filebeat(2c16g), 指标数据直接为beat metrics, es配置可忽略**

```
# bulk_max_size: The maximum number of events to bulk in a single Elasticsearch bulk API index request. The default is 50.
# The number of workers per configured host publishing events to Elasticsearch.
output.elasticsearch:
  bulk_max_size: 3200
  worker: 2
```
* 调整mem queue(消耗更多mem, 减少实时性)
    * queue.mem.event = 2 * workers * batch size
    * queue.mem.flush.min_events = batch size

* 当Throughput打满，考虑增加event compression
    `compression_level: 9`

* 猜想？
    * 能否通过增加filebeat instance, make 收集相同文件？

## 调整配置
### 增加filebeat性能监控
* 6.1.1不能使用新版本功能，通过自行metrics log收集，做展示；




* 7.4 查看metrics api使用方式
    在测试k8s增加 monitortarget 
