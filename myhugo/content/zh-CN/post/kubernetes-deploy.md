---
title: "Kubernetes中Deployment DaemonSet StatefulSet"
date: 2022-05-22T16:00:02+08:00
tags:
   - kubernetes
   - cloudnative
   - deployment 
   - daemonset 
categories:
   - kubernetes
   - cloudnative
   - deployment 
   - daemonset 
toc: true
---

`kubernetes`集群有几种部署和管理POD的工具，下面简单介绍一下。

### ReplicaSet
`ReplicaSet`是支持基于集合的标签选择器的下一代`Replication Controller`(新版本已不再使用)，
它主要用作`Deployment`协调创建、删除和更新`Pod`，和`Replication Controller`唯一的区别是，`ReplicaSet`支持标签选择器。
在实际应用中，虽然`ReplicaSet`可以单独使用，**但是一般建议使用`Deployment`来自动管理`ReplicaSet`**，
除非自定义的Pod不需要更新或有其他编排等。
```shell
## 查看RellicaSet资源
$ kubectl get rs
```

### Deployment
根据声明的`YAML`文件信息部署`POD`，并将`POD`调度到资源占用少的`node`节点上，会调用`ReplicaSet`管理`POD`副本。
主要适合如下场景：
1. 部署无状态应用
2. 管理Pod和ReplicaSet
3. 部署，滚动升级
4. 弹性扩容等
看一下`kubernetes`官方提供`deployment yaml`资源文件 
```shell
apiVersion: apps/v1
kind: Deployment          ## 资源类型
metadata:
  name: nginx-deployment  ## 名称
  labels:
    app: nginx            ##标签
spec:
  replicas: 3             ## 副本数量
  selector:
    matchLabels:
      app: nginx          ## deployment根据这个标签，选择管理POD
  template:
    metadata:
      labels:
        app: nginx       ## Pod的标签，跟selector保持一致
    spec:
      containers:
      - name: nginx         ## 容器镜像
        image: nginx:1.14.2 ## 镜像版本
        ports:
        - containerPort: 80 ## 容器的端口
```

### DaemonSet
`DaemonSet`(守护进程集)和守护进程类似，它在符合匹配条件的`kubernetes`集群的node节点上均部署一个Pod。
`DaemonSet`的场景:
1. 集群存储守护程序，如`glusterd`、`ceph`要部署在每个节点上提供持久性存储。
2. 集群日志守护程序，如`fluentd`、`logstash`，在每个节点运行容器。
3. 节点监视守护进程，如`prometheus`监控集群，可以在每个节点上运行一个`node-exporter`用进程来收集监控节点的信息。

```shell
apiVersion: apps/v1
kind: DaemonSet     ## 资源类型
metadata:
  name: fluentd-elasticsearch
  namespace: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
      # 这些容忍度设置是为了让守护进程在控制平面节点上运行
      # 如果你不希望控制平面节点运行 Pod，可以删除它们
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes:
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

### StatefulSet
`StatefulSet`主要用于管理有状态应用程序的工作负载`API`对象。
比如在生产环境中可以部署`ElasticSearch`集群、`MongoDB`集群或者需要持久化的`RabbitMQ`集群、
`Redis`集群、`Kafka`集群和`ZooKeeper`集群等。和`Deployment`类似，一个`StatefulSet`也同样管
理着基于相同容器规范的`Pod`。不同的是`StatefulSet`为每个`Pod`维护了一个粘性标识。
这些`Pod`是根据相同的规范创建的，但是不可互换，每个Pod都有一个持久的标识符，在重新调度时也会保留，
一般格式为`StatefulSetName-Number`。比如定义一个名字是`Redis-Sentinel`的`StatefulSet`，指定创建三个`Pod`，
那么创建出来的`Pod`名字就为`Redis-Sentinel-0`、`Redis-Sentinel-1`、`Redis-Sentinel-2`。
而`StatefulSet`创建的`Pod`一般使用`Headless Service`(无头服务)进行通信。

```shell
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # 必须匹配 .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # 默认值是 1
  minReadySeconds: 10 # 默认值是 0
  template:
    metadata:
      labels:
        app: nginx # 必须匹配 .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```


### 引用
1. [Deployment](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)

2. [DaemonSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/)

3. [StatefulSet](https://kubernetes.io/zh/docs/concepts/workloads/controllers/statefulset/)

4. [资源调度-Deployment，StatefulSet，DaemonSet](https://www.yj-example.cn/?p=925)
