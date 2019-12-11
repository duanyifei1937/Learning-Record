# 定制化日志收集

filebeat --> kafka --> logstash --> ES/localfile 

## Filebeat
[配置文件参考](duanyifei1937_Kubernetes-ansible_ kubernetes all)
* stdout & redirect 都要收集
* **定制pod是否需要收集日志(annotation)(drop-event)**；
* 对接kafka topic如何定义？
* **延时问题**
* filebeat 增加 `service` field, 用来发送到不同kafka topic;

## kafka
* ~~~

## Logstash
* message timestamp overwrite;
* 落盘文件(格式)
* 区分k8s log and 非k8s log, 统一拉取alert;





