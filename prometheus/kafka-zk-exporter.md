# Kafka Zookeeper Exporter

# prometheus kafka 监控
* kafka监控之前使用[jmxtrans+InfluxDb+Grafana](http://blog.51cto.com/navyaijm/1958376), 每次增加消费者，需要手动修改脚本

## kafka exporter
> 之前使用(jmxtrans+InfluxDb+Grafana)实现kafka监控及报警(不推荐：grafana做告警渠道，使用prometheus实现统一化)；
> prometheus监控kafka, 使用组件：kafka exporter && jmx exporter
> 取消报警需要手动修改kafka scripts问题；

### jmx_exporter
* 使用supervisor管理kafka进程：

```
[program:kafka]
environment = JMX_PORT = 9999
environment = KAFKA_OPTS = "-javaagent:/data/prometheus/jmx_exporter/jmx_prometheus_javaagent-0.3.1.jar=9990:/data/prometheus/jmx_exporter/kafka-agent.yaml"
command = /data/soft/kafka_2.11-2.0.0/bin/kafka-server-start.sh /data/soft/kafka_2.11-2.0.0/config/server.properties
directory = /data/soft/kafka_2.11-2.0.0
startsecs = 5 ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true ; 程序异常退出后自动重启
startretries = 3 ; 启动失败自动重试次数，默认是 3
user = work ; 用哪个用户启动
redirect_stderr = true ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20 ; stdout 日志文件备份数
stdout_logfile = /data/logs/kafka/supervisor.log
```

* kafka-agent.yaml
(此处注释hostport & jmxurl,默认使用本地jmx端口)

```
---
startDelaySeconds: 0
#hostPort: 172.16.16.54:9999
#jmxUrl: service:jmx:rmi:///jndi/rmi://172.16.16.54:9999/jmxrmi
ssl: false
lowercaseOutputName: false
lowercaseOutputLabelNames: false
whitelistObjectNames: ["org.apache.cassandra.metrics:*"]
blacklistObjectNames: ["org.apache.cassandra.metrics:type=ColumnFamily,*"]
rules:
  - pattern: 'org.apache.cassandra.metrics<type=(\w+), name=(\w+)><>Value: (\d+)'
    name: cassandra_$1_$2
    value: $3
    valueFactor: 0.001
    labels: {}
    help: "Cassandra metric $1 $2"
    type: GAUGE
    attrNameSnakeCase: false
```

* grafana：(推荐：3066)
* 根据特定添加alert rules;

## kafka_exporter

```
[program:kafka_exporter]
command = /data/prometheus/kafka_exporter/kafka_exporter --kafka.server=172.16.16.54:9092
directory = /data/prometheus/kafka_exporter/
startsecs = 5 ; 启动 5 秒后没有异常退出，就当作已经正常启动了
autorestart = true ; 程序异常退出后自动重启
startretries = 3 ; 启动失败自动重试次数，默认是 3
user = work ; 用哪个用户启动
redirect_stderr = true ; 把 stderr 重定向到 stdout，默认 false
stdout_logfile_maxbytes = 20MB ; stdout 日志文件大小，默认 50MB
stdout_logfile_backups = 20 ; stdout 日志文件备份数
stdout_logfile = /data/logs/kafka_exporter/supervisor.log
```

* grafana: (推荐：7589)

## grafana
* 根据情况绘制grafana doshboard:
![](https://raw.githubusercontent.com/duanyifei1937/Picture-bed/master/blog-img/kafka1.png)
![](https://raw.githubusercontent.com/duanyifei1937/Picture-bed/master/blog-img/kafka2.png)


## 参考
http://blog.51cto.com/navyaijm/1958376
https://github.com/prometheus/jmx_exporter
https://github.com/danielqsj/kafka_exporter#compatibility

