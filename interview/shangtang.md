# shangtang - 20191113

在`shangtang`面到了第四面，然后没有然后了，最难的问题，其实也是最简单的问题 -- “你未来三到五年的规划是什么样的？”

是啊，一个对自己都没有规划的人，能和公司发展、需要符合吗？单纯就为了混口饭吃来吗？

那这个问题究竟应该如何回答？ 自己的心里想法究竟是什么样呢？

好难回答
也重来没有想过要回答这样的问题，但这个恰恰是最关键的。

现在来写一下吧，自己的真实想法：

1. 从来没有感觉这一把面了几家，全部能面到最后很牛；自己几斤几两还是很清楚的；用到的没有用好，缺陷地方多得是，所以两个字概括 -- “深入”；
2. 更多的能自己写一些工具，可以有patch能力;

## 一些问题及解答
* k8s log collection:
    * k8s redirect log (两个pod同时漂在一个node上面，mount相同文件，锁问题)；
    * stdout & stderr & 业务日志一起打入前台输出收集；
    * Fluentd使用；
* 灰度 - ingress (canary & mirror)原理；
* ansible -- 原理、自定义module;
* 网络：
    * 网络打通， 网卡 --> docker0 or vni0?
    * 如何访问内部pod ip?
* ingress 全局配置？ annotation or comfigmap?
* Rancher的使用？
* prometheus-operator的增加业务libnary, monitor target;
    * 给ep增加monitor target;
    * 集群外信息注入集群内部监控；

---
逐步简答~
