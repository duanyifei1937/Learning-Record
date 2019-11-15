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
        * (会存在lock的问题, 这样使用会造成miss log or lines being cut)
        * [how kubernetes deal with file write locker accross multi pods when hostpath Volumes concerned](https://stackoverflow.com/questions/52052573/how-kubernetes-deal-with-file-write-locker-accross-multi-pods-when-hostpath-volu)
        * 现在的使用方式是有问题的！~~
    * stdout & stderr & 业务日志一起打入前台输出收集；
        * 这样做的一些弊端：
            * 业务日志与启动日志混合；
            * 真实业务场景，一个pod生成多种日志，混合到一处不方便管理；
    * Fluentd使用；
        * 使用后介绍~
* 灰度 - ingress (canary & mirror)原理；
    * canary: 类似于做multip-idc 流量切换的过程，按一定比例，将`$host`改写，流量访问到修改后的upstream;
    * mirror: 借助nginx mirror module, 针对特定路径，`mirroring of an original request by creating background mirror subrequests. Responses to mirror subrequests are ignored.`

* ansible -- 原理、自定义module;
* 网络：
    * 网络打通， 网卡 --> docker0
        * docker0相当于一张网卡，在node上增加路由信息，匹配到pod ip ranger路由规则匹配;
    * 如何访问内部pod ip??
* ingress 全局配置？ annotation or comfigmap?
    * configmap 用来做全局配置nginx配置信息；
    * annotation domain级别；
    * template configmap无法实现的时候；
* Rancher的使用？
* prometheus-operator的增加业务libnary, monitor target;
    * 给ep增加monitor target;
    * 集群外信息注入集群内部监控；

---
逐步简答~
