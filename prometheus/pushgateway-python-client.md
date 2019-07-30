# Pushgateway Python Client
> 有时需要自定义metrics(比如从ksyun控制台拉取VPC NAT带宽信息)，进行阈值判断报警，介绍metrics注入方式；

prometheus-pushgateway: 1.2.3.4:9091

* 实现label可动态增加

```
from prometheus_client import CollectorRegistry, Gauge, pushadd_to_gateway
import socket
import time


def report(job='', metric='', desc='', val='', **labels):
    # definition instance
    # endpoint = "video:" + socket.gethostname()
    # ts = int(time.time())

    registry = CollectorRegistry()
    if len(labels) > 0:
        labelname = labels.keys()
        g = Gauge(metric.replace('.', '_'), desc, labelnames=labelname, registry=registry)
        lastpush = Gauge('lastpush_' + metric.replace('.', '_'), desc, labelnames=labelname, registry=registry)

        g.labels(**labels).set(val)
        lastpush.labels(**labels).set_to_current_time()

        grouping_key = labels
        pushadd_to_gateway('1.2.3.4:9091', job=job, grouping_key=grouping_key, registry=registry)

    else:
        g1 = Gauge(metric.replace('.', '_'), desc, registry=registry)
        g1.set(int(val))
        g2 = Gauge('lastpush_' + metric.replace('.', '_'), desc, registry=registry)
        g2.set_to_current_time()
        pushadd_to_gateway('1.2.3.4:9091', job=job, registry=registry)


if __name__ == '__main__':
    # 可自定义labels
    report('JobA', 'metrics_test111', 'desc_test111', 444, a1=1, a2=2)
```

