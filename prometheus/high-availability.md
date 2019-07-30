# High-Availlability
> 介绍prometheus out of k8s cluster监控方式。

![xx](https://raw.githubusercontent.com/duanyifei1937/Picture-bed/master/blog-img/prometheus-avaliable.png)
如图:
* 非k8s节点服务注册使用三台consul,注册信息同步及高可用；
* prome 分别两台同时获取consul + k8s metrics;
* alertmanager 组成cluste, 两者使用gossip同步alert notify,实现inhibit;
* alert 发送至ngx, 由upstream剔除故障Ding-webhook(**增加subrouteing 升级规则, in feature**);
* 为高可用，部署两台Prometheus and Alertmanager, 但每台拉取数据的时间戳不一致，会产生脏数据，如何解决？ alertmanager 如何同步notification, 做到同步与inhibit;
* two prometheus:
    * In addition the two Prometheus servers have slightly different external labels, so their data does not conflict if remote storage is in use. We then use alert relabelling to ensure they still send identically labelled alerts, which the Alertmanager will automatically de-duplicate.

``` yaml
# On a machine named "prom-1":
wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-*.linux-amd64.tar.gz
cd prometheus-*
cat > prometheus.yml << EOF
global:
  external_labels:
    dc: europe1    
alerting:
  alert_relabel_configs:
    - source_labels: [dc]
      regex: (.+)\d+
      target_label: dc
  alertmanagers:
    - static_configs:
      - targets: ['am-1:9093', 'am-2:9093']
# The rest of your Prometheus config goes here as usual.
EOF


# On a machine named "prom-2":
wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-*.linux-amd64.tar.gz
cd prometheus-*
cat > prometheus.yml << EOF
global:
 external_labels:
   dc: europe2   # Note that this is different only by the trailing number.
alerting:
 alert_relabel_configs:
 - source_labels: [dc]
   regex: (.+)\d+
   target_label: dc
 alertmanagers:
 - static_configs:
   - targets: ['am-1:9093', 'am-2:9093']
# The rest of your Prometheus config goes here as usual.
EOF
```

* two alertmanager:
    * alertmanager cluster support high availability, use [--cluster.*](https://github.com/prometheus/alertmanager#high-availability)
    * alertmanager 组成cluste, 两者使用gossip同步alert notify,实现inhibit;

## reference:
[High Availability Prometheus Alerting and Notification](https://www.robustperception.io/high-availability-prometheus-alerting-and-notification)