# etcd

## etcdserver space quota
* 报错
`etcdserver: mvcc: database space execeded`

* 原因
    * Storage size limit[Storage size limit](https://github.com/etcd-io/etcd/blob/master/Documentation/dev-guide/limit.md)
The default storage size limit is 2GB, configurable with --quota-backend-bytes flag. 8GB is a suggested maximum size for normal environments and etcd warns at startup if the configured value exceeds it.

* 恢复
[压缩数据并碎片回收](https://doczhcn.gitbook.io/etcd/index/index-1/maintenance#li-shi-ya-suo)

* 调优：
    * 增加`--auto-compaction-retention`, 并定期碎片回收；

## etcd模拟node故障
* 概况
三节点的etcd集群，将其中一台etcd daemon stop, data_dir 并清除数据路径/var/lib/etcd/；
* 集群中出现unhealth节点

```
$ etcdctl cluster-health
member 8f89baa11f8bea5b is healthy: got healthy result from http://172.16.16.27:2379
member e3e85ebde6758575 is healthy: got healthy result from http://172.16.16.33:2379
```

* delete unhealth member

```
etcdctl delete member xxx
```

* 新加节点

```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd
EnvironmentFile=-/data/soft/k8s_deploy/kubernetes/etcd.conf
ExecStart=/usr/local/bin/etcd \
  --name ${ETCD_NAME} \
  --initial-advertise-peer-urls ${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
  --listen-peer-urls ${ETCD_LISTEN_PEER_URLS} \
  --listen-client-urls ${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
  --advertise-client-urls ${ETCD_ADVERTISE_CLIENT_URLS} \
  --initial-cluster-token ${ETCD_INITIAL_CLUSTER_TOKEN} \
  --initial-cluster etcd1=http://172.16.16.27:2380,etcd2=http://172.16.16.33:2380,etcd3=http://172.16.16.14:2380 \
  --initial-cluster-state existing \
  --data-dir=${ETCD_DATA_DIR}
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

**--initial-cluster-state existing**


* 加入新成员
    * 报错

```
etcd: error validating peerURLs {ClusterID:194cd14a48430083 Members:[&{ID:1ce6d6d01109192 RaftAttributes:{PeerURLs: 
[https://192.168.0.102:2380]} Attributes:{Name:etcd03 ClientURLs:[https://192.168.0.102:2379]}} &{ID:9b534175b46ea789 RaftAttributes:{PeerURLs:[https://192.168.0.100:2380]} 
Attributes:{Name:etcd01 ClientURLs:[https://192.168.0.100:2379]}}] RemovedMemberIDs:[]}: member count is unequal
```

* 解决
    `etcdctl member add etcd3 http://172.16.16.14:2380`
    
    * etcd: the member has been permanently removed from the cluster the data-dir used by this member must be removed


## 增加IO优先级
* [磁盘IO优先级](https://www.rancher.cn/docs/rancher/v2.x/cn/install-prepare/best-practices/etcd/#%E4%BA%8C-%E7%A3%81%E7%9B%98io%E4%BC%98%E5%85%88%E7%BA%A7)

## follower 增加网络带宽优先级
* [网络延迟](https://www.rancher.cn/docs/rancher/v2.x/cn/install-prepare/best-practices/etcd/#%E5%9B%9B-%E7%BD%91%E7%BB%9C%E5%BB%B6%E8%BF%9F)
