# banyu

## filebeat collection node log 
* Q: 服务pod通过hostpath mount node /data/logs, 将日志重定向到node, filebeat统一收集；
如果业务pod重启，log文件如何处理？ 追加？ 新建？

* A: 测试结果如果pod重启，日志追加;

## k8s调优
etcd & kubelet
### kubelet

| 参数 | Fast | Default | Medium | Low | 含义 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| --node-status-update-frequency | 4s | 10s | 20s | 1m | kubelet上传自身状态 |
| --node-monitor-period | 2s | 5s | 5s | 5s | kube-controller多久查看一次node状态 |
| --node-monitor-grace-period | 20s | 40s | 2m | 5m | kube-controller认为node多久无响应后是NotReady |
| --pod-eviction-timeout | 30s | 5m | 5m | 1m | 认为node无响应后多久驱逐该node上的pod |

### kube-scheduler
* multip-idc array 自定义调度策略

### DNS
无效解析问题 [Kubernetes内部域名解析原理](https://ccnuo.com/2019/08/25/CoreDNS%EF%BC%9AKubernetes%E5%86%85%E9%83%A8%E5%9F%9F%E5%90%8D%E8%A7%A3%E6%9E%90%E5%8E%9F%E7%90%86%E3%80%81%E5%BC%8A%E7%AB%AF%E5%8F%8A%E4%BC%98%E5%8C%96%E6%96%B9%E5%BC%8F/)
``` yaml
nameserver 10.233.0.3
search cicd.svc.cluster.local svc.cluster.local cluster.local
options ndots: 5
默认5
.的数量 < ndots:  search domain cluster
.数量  >= ndots:  outof cluster domain
```

## kube-proxy iptables 优化
* 链： svc --> ep 条目过多，顺序匹配，如何优化？
* ipset分组

## k8s alert 指标

## cpu、io、load高如何排查
