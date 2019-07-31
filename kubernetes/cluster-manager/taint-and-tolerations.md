# Taint And Tolerations
> 将pod固定到某台node,并禁止其他pod往上漂移；

``` yaml
# 为节点打taint label

kubectl taint nodes vm1-2-3-4.ksc.com usage=ranger-datajet-pg:NoSchedule

# deploy yaml 写法
# 如果指定的node noready, 也会往其他节点漂移，为防止，指定nodeselect固定节点；
nodeSelector:
  kubernetes.io/hostname: vm1-2-3-4.ksc.com
tolerations:
- effect: NoSchedule
  key: usage
  operator: Equal
  value: ranger-datajet-pg
```

* yaml example

``` yaml
spec:
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: tamers-zeppelin
    spec:
      containers:
      dnsPolicy: ClusterFirstWithHostNet
      hostNetwork: true
      nodeSelector:
        kubernetes.io/hostname: 10.0.0.1
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
      tolerations:
      - effect: NoSchedule
        key: usage
        operator: Equal
        value: comptuer
      volumes:
      - hostPath:
          path: /data/k8s/zeppelin
          type: DirectoryOrCreate
        name: tamers-data
```

* reference:
https://kubernetes.io/zh/docs/concepts/configuration/taint-and-toleration/