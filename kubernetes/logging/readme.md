# Log-Collection
> k8s日志收集方案

## 之前日志收集方式
* k8s daemonset部署filebeat固定收集 `/data/logs/*.log`;
* 业务pod挂载node `/data/logs`到容器内部，进行写操作；
* 日志命名方式： app-gw.20180102.log

* https://github.com/duanyifei1937/kubernetes-deploy/blob/master/log-collect/Filebeat/filebeat.yaml

* offical:

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
    filebeat.inputs:
    - type: container
      paths:
        - /var/log/containers/*.log
      processors:
        - add_kubernetes_metadata:
            host: ${NODE_NAME}
            matchers:
            - logs_path:
                logs_path: "/var/log/containers/"

    processors:
      #- add_cloud_metadata:
      - add_host_metadata:

    output.elasticsearch:
      hosts:
        - x.x.x.x:9200
        - x.x.x.x:9200
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat
  namespace: kube-system
  labels:
    k8s-app: filebeat
spec:
  selector:
    matchLabels:
      k8s-app: filebeat
  template:
    metadata:
      labels:
        k8s-app: filebeat
    spec:
      serviceAccountName: filebeat
      terminationGracePeriodSeconds: 30
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      tolerations:
      - operator: Exists
        effect: NoSchedule
      containers:
      - name: filebeat
        image: registry/k8s/elastic/filebeat:7.4.0
        args: [
          "-c", "/etc/filebeat.yml",
          "-e",
        ]
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        securityContext:
          runAsUser: 0
          # If using Red Hat OpenShift uncomment this:
          #privileged: true
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
        - name: config
          mountPath: /etc/filebeat.yml
          readOnly: true
          subPath: filebeat.yml
        - name: data
          mountPath: /usr/share/filebeat/data
        - name: varlibdockercontainers
          mountPath: /data/docker/containers
          readOnly: true
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: timezone
          mountPath: /etc/localtime
          readOnly: true
      volumes:
      # configmap config file:
      - name: config
        configMap:
          defaultMode: 0600
          name: filebeat-config
      # 实际存放containers的日志路径，根据docker data-root修改；
      - name: varlibdockercontainers
        hostPath:
          path: /data/docker/containers
      # /var/log为系统日志路径，挂载后，filebeat只collection containers的soft link;
      - name: varlog
        hostPath:
          path: /var/log
      # data folder stores a registry of read status for all files, so we don't send everything again on a Filebeat pod restart
      # 哇塞，filebeat自己他么提供了防止重复收集的registry挂载！~~ 牛逼啊
      - name: data
        hostPath:
          path: /data/filebeat-data
          type: DirectoryOrCreate
      - name: timezone
        hostPath:
          path: /etc/localtime

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
- apiGroups: [""] # "" indicates the core API group
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

## 存在问题
* 当multip pod在同一台node上时，存在write lock问题，会造成log miss and log cut;
* 当 pod crush, 日志文件还是追加到固定文件；(此处没有问题)
* 只能收集redirect log, 容器启动日志无法收集；
* 当统一收集`/var/lib/docker/containers/[container-id]/[container-id]-json.log`时:
    * 存在很多不需要收集的日志pod, 例如`pause pod` and `calico` ...etc.
    * 无法标识log label;
---
* 除此之外，做了更精细化的定制：
    * drop some fields;
    * drop event;
    * message decode;
    * ... etc.
## 修改后
* 依然使用`/var/logs/containers`方式，所有业务pod前台输出，统一收集；
* 使用filebeat **add_kubernetes_metadata**plugin功能，标识日志 label;
(原理是扫描docker container /var/lib/docker/container/*/*.log日志的同时,会通过container id去kubernetes api匹配对应的pod为每条日志打上kubernetes pod相关的标签,然后我们可以利用这些标签信息做一些日志处理)
* 配置示例

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
      hosts:
        - x.x.x.x:9092
        - x.x.x.x:9092
      version: 0.9.0.1
      topic: 'filebeat-kubernetes-xx'
      partition.round_robin:
        reachable_only: false
      compression: snappy
    spool_size: 2048
    idle_timeout: 5s
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
      # filebeat 7.0
      combine_partials: true
      containers.ids:
        - "*"
      encoding: plain
      fields_under_root: true
      ignore_older: 0
         close_older: 1h
      scan_frequency: 10s
      harvester_buffer_size: 16384
      max_bytes: 10485760
      tail_files: true
      backoff: 1s
      max_backoff: 10s
      backoff_factor: 2
      # comment,pod templateannotatioES.
      # filebeat.harvest: Only harvest pods that have a value of `true`
      # filebeat.index: Set the index value stored in elasticsearch
      processors:
        - add_kubernetes_metadata:
            in_cluster: true
            include_annotations:
              - "filebeat.harvest"
              - "filebeat.index"
        # comment(wangyichen): drop_eventdrop_fields drop_event kubernetes.annotations.
        - drop_event:
            when:
              not:
                equals:
                  kubernetes.annotations.filebeat.harvest: "true"
        - decode_json_fields:
            fields: ["message"]
            target: ""
            overwrite_keys: true
            when:
              regexp:
                message: "^{.*}$"
        - rename:
             fields:
               - from: "message"
                 to: "logs"
             when:
               not:
                 regexp:
                   message: "^{.*}$"
        - drop_fields:
            fields:
              - "beat"
              - "@metadata"
              - "kubernetes.labels"
              - "prospector"
              - "input"
              - "offset"
              - "stream"
              - "source"
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
      containers:
        - name: filebeat
          image: elastic/filebeat:6.3.1
          args: [
            "-c", "/etc/filebeat.yml","-e", ]
          securityContext:
            runAsUser: 0
          resources:
            limits:
              memory: 200Mi
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
      volumes:
        - name: config
          configMap:
            defaultMode: 0600
            name: filebeat-config
        - name: varlibdockercontainers
          hostPath:
            path: /data/data/docker/containers
        - name: inputs
          configMap:
            defaultMode: 0600
            name: filebeat-inputs
        # We set an `emptyDir` here to ensure the manifest will deploy correctly.
        # It's recommended to change this to a `hostPath` folder, to ensure internal data
        # files survive pod changes (ie: version upgrade)
        - name: data
          emptyDir: {}
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

* 业务pod使用方式：
    * filebeat.harvest: "true" #只有当此annotation 设置为“true”时,日志才会被采集
    * filebeat.index: "account" # 根据这个值定义我们在日志最终输出到ES中哪个索引,最终的名称是kubernetes-account-%{+YYYY.MM.dd}" 

    ``` yaml
    apiVersion: apps/v1
kind: Deployment
metadata:
  name: account
  namespace: base-platform
  labels:
    app.kubernetes.io/name: account
    app.kubernetes.io/component: base-platform
    app.kubernetes.io/part-of: account
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      product: base-platform
      app: account
      env: production
  template:
    metadata:
      annotations:
        filebeat.harvest: "true"
        filebeat.index: "account"
      labels:
        product: base-platform
        app: account
        env: production
    spec:
      imagePullSecrets:
        - name: docker-secret
      dnsConfig:
        options:
          - name: ndots
            value: '2'
          - name: timeout
    ```