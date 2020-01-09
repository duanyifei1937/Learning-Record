[toc]
# 保持顺序
> 日志收集各种情况下如何保持数据有序？

## 通常方式
filebeat --> kafka --> logstash --> es --> kibana
无论前几个组件是否保存有序，最终通过kibana搜索，search key为timestamp, 并不要求中间过程文件有序，可使用异步、并发；

但在某些情况下，比如需要output to file, 此时就需要考虑数据顺序保持；

## filebeat --> logstash --> file
需要配置：

``` yaml
logstash:
# The number of workers that will, in parallel, execute the filter and output stages of the pipeline. 
# If you find that events are backing up, or that the CPU is not saturated, 
# consider increasing this number to better utilize machine processing power. 
# default: Number of the host’s CPU cores
pipeline.workers: 1
```

* filebeat to logstash 存在以下配置

``` yaml
filebeat:
#Configures number of batches to be sent asynchronously to logstash while waiting for ACK from logstash. 
#Output only becomes blocking once number of pipelining batches have been written. 
#Pipelining is disabled if a value of 0 is configured. The default value is 2.
pipelining: 2
```
任务此处也应该配置为1， 但通过测试并不需要，可能没有压测到filebeat wait logstash ack的情况；


## filebeat --> kafka --> logstash --> file
> 提供两种方式

### single partition(在topic级别做了拆分)
topic使用single partition, 保证顺序性；

### key-ordering(消息键保序策略)
生成者往kafka写入时，具体往哪个partiton写入，有三种策略：`random` & `round_robin` & `hash`;
不同source file, 定义不同key, 一旦消息被定义了 Key，就可以保证同一个 Key 的所有消息都进入到相同的partiton里面，由于每个分区下的消息处理都是有顺序的;
单一消费者只能消费一个partition;
在根据source-path落盘到不同文件；

* medadata:

```
{"input":{"type":"log"},"host":{"name":"cn-hz-wl-test-k8s-01"},"message":"94","ecs":{"version":"1.1.0"},
"@version":"1","@timestamp":"2019-12-18T13:13:45.166Z","log":{"file":{"path":"/usr/share/filebeat/logs/ddd.log"},"offset":270},
"agent":{"type":"filebeat","id":"61b3d6cf-488b-4197-a50b-9d76522c5cf4","version":"7.4.2","hostname":"cn-hz-wl-test-k8s-01","ephemeral_id":"7b7be355-90b7-4004-be33-e720e7a05dff"}}
```


``` yaml
filebeat.inputs:
- type: log
  paths:
    - /usr/share/filebeat/logs/aaa.log
  fields:
    key: aaa

- type: log
  paths:
    - /usr/share/filebeat/logs/bbb.log
  fields:
    key: bbb

output.kafka:
    hosts: ["aaa:9092","bbb:9092","ccc:9092"]
    topic: 'dyf-test'
    key: "%{key}"
```

### 两种方式的对比
* 虽都可以保证顺序性，但single partition丧失了kafka多分区带来的高吞吐、负载均衡优势；第二种方式本质上是topic细粒度的拆分，比如根据业务唯一ID等；

## 测试环境修改
* log收集流程：
docker stdout --> filebeat --> logstash --> file

* 修改:
    * logstash config:
    
    ``` yaml 
    pipeline.workers: 1
    ```


## 线上环境修改
* log收集流程：

``` yaml
docker stdout --> log-pilot --> kafka --> logstash --> file
```
为避免减少partiton操作(删除重建，采用第二种方式)

* 修改:
    * log-pilot
        * 拆分(stdout & redirect log 分离)
        * 增加output kafka key配置；
    
    * logstash:
        * single instance;
        * modify config: `pipeline.workers: 1`
        * input from kafka `consumer_threads` (一个消费线程只消费单个partition内数据，不必要必须为1；)
    * kafka统一为一个topic;

* 新增加filebeat只收集k8s stdout:

``` yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
  namespace: kube-system
  labels:
    k8s-app: filebeat
data:
  filebeat.yml: |-
    filebeat.config:
      inputs:
        # Mounted `filebeat-inputs` configmap:
        path: /usr/share/filebeat/inputs.d/*.yml
        # Reload inputs configs as they change:
        reload.enabled: true
      modules:
        path: /usr/share/filebeat/modules.d/*.yml
        # Reload module configs as they change:
        reload.enabled: true

    output.kafka:
      hosts: ["10.111.203.9:9092","10.111.203.10:9092","10.111.203.13:9092"]
      topic: 'beat-k8sall-stdout'
      worker: 3
      key: %{[kubernetes][pod][name]}

    spool_size: 2048
    idle_timeout: 5s
    logging.metrics.enabled: true
    logging.metrics.period: 10s
    http.host: 0.0.0.0
    http.port: 5066
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-inputs
  namespace: kube-system
  labels:
    k8s-app: filebeat
data:
  kubernetes.yml: |-
    - type: docker
      # filebeat 7.0以后支持
      combine_partials: true
      containers.ids:
        - "*"
      encoding: plain
      fields_under_root: true
      ignore_older: 0
      scan_frequency: 10s
      harvester_buffer_size: 163840
      max_bytes: 104857600
      tail_files: true
      backoff: 1s
      max_backoff: 10s
      backoff_factor: 2
      close_inactive: 48h
      processors:
        - add_kubernetes_metadata:
            in_cluster: true
            include_annotations:
              - "filebeat.harvest"
              - "filebeat.index"
        - drop_event:
            when:
              not:
                equals:
                  #kubernetes.annotations.filebeat.harvest: "true"
                  kubernetes.namespace: "service"
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: filebeat
  namespace: kube-system
  labels:
    k8s-app: filebeat
spec:
  template:
    metadata:
      labels:
        k8s-app: filebeat
    spec:
      serviceAccountName: filebeat
      terminationGracePeriodSeconds: 30
      nodeSelector:
        role: service
      containers:
        - name: filebeat-exporter
          image: hub.pri.ibanyu.com/library/beat-exporter:latest
        - name: filebeat
          image: hub.pri.ibanyu.com/library/filebeat:6.3.1
          args: [
            "-c", "/etc/filebeat.yml",
            "-e",
          ]
          securityContext:
            runAsUser: 0
          resources:
            limits:
              memory: 1000Mi
              cpu: "3"
            requests:
              cpu: 100m
              memory: 100Mi
          volumeMounts:
            - name: config
              mountPath: /etc/filebeat.yml
              readOnly: true
              subPath: filebeat.yml
            - name: inputs
              mountPath: /usr/share/filebeat/inputs.d
              readOnly: true
            - name: data
              mountPath: /usr/share/filebeat/data
            - name: varlibdockercontainers
              mountPath: /var/lib/docker/containers
              readOnly: true
            - name: stdout
              mountPath: /data/logs/service
      volumes:
        - name: config
          configMap:
            defaultMode: 0600
            name: filebeat-config
        # 需要修改:
        - name: varlibdockercontainers
          hostPath:
            path: /data/apps/docker/containers
        - name: inputs
          configMap:
            defaultMode: 0600
            name: filebeat-inputs
        # We set an `emptyDir` here to ensure the manifest will deploy correctly.
        # It's recommended to change this to a `hostPath` folder, to ensure internal data
        # files survive pod changes (ie: version upgrade)
        - name: data
          emptyDir: {}
        # 可删除
        - name: stdout
          hostPath:
            path: /data/logs/service
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: filebeat
subjects:
  - kind: ServiceAccount
    name: filebeat
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: filebeat
  apiGroup: rbac.authorization.k8s.io
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: filebeat
  labels:
    k8s-app: filebeat
rules:
  # "" indicates the core API group
  - apiGroups: [""]
    resources:
      - namespaces
      - pods
    verbs:
      - get
      - watch
      - list
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: filebeat
  namespace: kube-system
  labels:
    k8s-app: filebeat
---
```

* logstash修改：

``` yaml
 # 修改logstash落盘配置
 kafka {
    bootstrap_servers => "10.111.203.9:9092,10.111.203.10:9092,10.111.203.13:9092"
    group_id => "logstash-panic-stdout"
    auto_offset_reset => "latest"
    consumer_threads => 3
    topics_pattern => "beat-k8sall-stdout"
    codec => json
  }
```