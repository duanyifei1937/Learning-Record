# Logging
> 介绍k8s集群日志管理

[toc]
## Log Kind(k8s产生日志的三种类型)
* Containers stdout & stderr
    * (容器启动时的stdout and stderr，被`logging-driver` redirect 到`/var/lib/docker/containers/[container-id]/[container-id]-json.log`)
    * 可通过`docker logs -f xx`查看
    * 切割与保留：`/etc/docker/daemon.json` `log-opts`

* Containers Redirect Log
    * mount node path;
    * 通过`Daemonset`部署`filebeat or fluentd`收集；
    * 日志切割: 程序写入自定义(目前以天为单位)；
    * 日志保留： `Daemonset`部署`clean-log`实现

* Component Log
    * systemd --> jourend(会造成和`/var/log/message`重复收集问题)
    * 切割&保留： logrotated(3d)
(本质上第一种和第二种相同)

## Log Collection Kind(日志收集的集中方式)
* node level: daemonset pod
* pod level: sidecar(ingress-nginx-controller)

## Log Collection OpenSource way
* Filebeat + kafka + logstash + Elasticsearch + Kibana
* Fluent Bit + anyother

### ELK
#### Filebeat
* filebeat使用`DaemonSet`收集all node `${Docker Root Dir}/containers`系统启动日志(存在multip-line问题)
* 可定制收集某目录下业务程序重定向日志`/data/logs/apps/*.log`
* note: `taint & tolerations` unschuduler node 也部署；
* note: **app pod timezone问题， mount sys localtime**;
* note: 权衡 `log-opts` 与 `写入延时高`的问题;

#### Elasticsearch
* 优化~


### EFK

## Reference
* [Running Filebeat on Kubernetes](https://www.elastic.co/guide/en/beats/filebeat/7.4/running-on-kubernetes.html)
* [K8s Logging Architecture](https://kubernetes.io/docs/concepts/cluster-administration/logging/#logging-at-the-node-level)
* [Logging Solutions For Kubernetes](https://medium.com/minerva-io/logging-solution-for-kubernetes-32490b8e0378): logging & monitor的开源项目；
* [Logging Best Practices for Kubernetes using Elasticsearch, Fluent Bit and Kibana](https://itnext.io/logging-best-practices-for-kubernetes-using-elasticsearch-fluent-bit-and-kibana-be9b7398dfee)
* [containers define timezone by self](https://serverfault.com/questions/683605/docker-container-time-timezone-will-not-reflect-changes)