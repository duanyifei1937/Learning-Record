# Prometheus Monitor GPU
> Prometheus 未提供GPU Export, 采用pushgateway自己实现；

* k8s中使用GPU，每一个pod只能看到自己使用的单独的gpu卡信息，因此将脚本部署到node节点，定时获取gpu使用率、gpu显存、温度 etc. 进行上传到prometheus;

* 参考 [github-gpu-monitor](https://github.com/duanyifei1937/gpu_monitor)