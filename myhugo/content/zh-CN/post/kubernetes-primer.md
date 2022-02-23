---
title: "Kubernetes入门"
date: 2022-02-22T09:43:17+08:00
tags:
   - kubernetes
   - k8s
   - cloudnative
categories:
   - kubernetes 
toc: true
---

> 以容器(docker)和容器编排(kubernetes)的云原生技术栈在后端开发中越来越重要，每个技术童鞋都有必要熟悉这个技术栈

# kubernetes入门

## kubernetes的核心组件
![arch](./components-of-kubernetes.svg)

### API Server

API server 的核心功能是提供k8s各类资源对象(如Pod、RC、Service)的增删改查及Watch等HTTP REST接口，成为集群内各个功能模块之间数据交互和通信的中心枢纽，是整个集群的数据总线和数据中心，运行在master节点。 
通常还具有以下功能。

- 集群管理的API入口
- 资源配额控制的入口
- 提供了完备的集群安全机制。

通常我们会通过kubectl命令与API server进行交互，提供restful API，所以说也可以通过代码方式直接调用k8s的API server。

### 控制器管理器(controller-manager)
controller-manager作为集群内部的管理控制中心，负责集群内部的Node、Pod、Endpoint、Namespace、ServiceAccount、ResourceQuota等的管理，意为控制器，运行在master节点。

- ReplicaSet Controller(副本控制器): 管理控制 pod 副本（服务集群）的数量，以使其永远与预期设定的数量保持一致。
- Endpoint Controller(节点控制器): Endpoint用来表示kubernetes集群中Service对应的后端Pod副本的访问地址，Endpoint Controller则是用来生成和维护Endpoints对象的控制器，其主要负责监听Service和对应Pod副本变化。
- Deployment Controller(部署控制器): Deployment中文意思为部署、调度，通过Deployment我们能操作RS（ReplicaSet）
- StatefulSet Controller(状态控制器): StatefulSet的出现是K8S为了解决 “有状态” 应用落地而产生的，Stateful这个单词本身就是“有状态”的意思 
- DaemonSet Controller(收回控制器): Daemon本身就是守护进程的意思，那么很显然DaemonSet就是K8S里实现守护进程机制的控制器
- Job Controller(任务控制器): 在K8S里运行批处理任务我们用Job即可
- CronJob Controller(cronjob控制器): 定时任务

### 调度器(scheduler)

kube-scheduler意为调度器，在集群承担了"承上启下"的重要功能，“承上”指的是它负责接收 Controller -manager创建的新Pod。为其安排一个可以安置的node;“启下”指的是安置完成之后，目前Node上的kubelet服务进程接管后继续工作，负责Pod生命周期中的下半生。

### kubelet

一个在集群中每个节点(node)上运行的代理。 它保证容器(containers)都 运行在 Pod 中。 kubelet 接收一组通过各类机制提供给它的 PodSpecs，确保这些 PodSpecs 中描述的容器处于运行状态且健康。

### kube-proxy

kube-proxy 是集群中每个节点上运行的网络代理， 实现 Kubernetes 服务(Service) 概念的一部分。

### etcd
    
存储数据

## kubernetes的请求流程
![arch](./k8s-arch.jpg)

### Pod

- Pod可以理解为是一组功能相同的容器，装的是docker创建的容器，也就是用来封装容器的一个容器；
- Pod 是一个虚拟化分组，有自己的IP地址和主机名hostname，利用namespace 进行资源隔离，相当于一台独立沙箱环境；
- Pod 相当于一台独立主机，内部可以封装一个或多个容器(通常是一组相关的容器)，内部容器之间访问采用 localhost。

![arch](./pod.png)

> Pod可以理解为豌豆荚

### Service

Kubernetes Service定义了逻辑上的一组Pod的抽象，可以通过一定策略访问这个Service, 我们一般称为微服务。 Service所针对的Pods集合通常是通过选择算符(标签选择器)来确定的，
Pod经常被创建和销毁，IP地址不固定，Service定义一组逻辑上的一组Pod的抽象，通过kube-proxy自动分配一个集群内部可以访问的虚IP，称为cluster IP。

在 Kubernetes 中，可以通过 Cluster Ip 来找到 Pod 的访问规则，但是 Cluster Ip 不好记啊，所以 Kuberbetes 提供了一个 CoreDns 的组件来对 Service 进行解析。
CoreDns 是一个DNS服务器，每当有Service创建时，都会在DNS服务里面增加一条记录。集群中的Pod可以通过`<SERVICE_NAME>.<NAMESPACE_NAME>`访问Service
```shell
dig codereviewapi

Server:         10.96.0.10
Address:        10.96.0.10:53

Name:   codereviewapi.codereview.svc.cluster.local
Address: 10.111.72.52
```

Kubernetes Service对外暴露服务通过NodePort、LoadBalancer和ExternalName

- NodePort：建立在ClusterIP类型之上，其在每个Node的IP地址的某静态端口（NodePort）暴露服务，NodePort的路由目标为ClusterIP，简单来说，NodePort类型就是在工作节点的IP地址上选择一个端口用于将集群外部的用户请求转发至目标Service的ClusterIP和Port，这种类型的Service既可如ClusterIP一样受到集群内部客户端Pod的访问，也会受到集群外部客户端通过套接字NodeIP:NodePort进行的请求；

- LoadBalancer：建构在NodePort类型之上，其通过cloud provider提供的负载均衡器将服务暴露到集群外部，LoadBalancer类型的Service会指向关联至Kubernetes集群外部的某个负载均衡设备，该设备通过工作节点之上的NodePort向集群内部发送请求流量，这种Service的优势在于，能够把来自于集群外部客户端的请求调度至所有节点（或部分节点）的NodePort之上，而不是依赖于客户端自行决定连接至哪个节点，从而避免了因客户端指定的节点故障而导致的服务不可用；

- ExternalName：通过将Service映射至由externalName字段的内容指定的主机名来暴露服务，此主机名需要被DNS服务解析至CNAME类型的记录。这种类型并非定义由Kubernetes集群提供的服务，而是把集群外部的某服务以DNS CNAME记录的方式映射到集群内，从而让集群内的Pod资源能够访问外部的Service的一种实现方式，这种类型的Service没有ClusterIP和NodePort，也没有标签选择器用于选择Pod资源。


### Ingress

Service对集群之外暴露服务的主要方式有两种：NodePort和LoadBalancer，但是这两种方式，都有一定的缺点：

- NodePort方式的缺点是会占用很多集群机器的端口，那么当集群服务变多的时候，这个缺点就愈发明显。

- LoadBalancer的缺点是每个Service都需要一个LB，浪费，麻烦，并且需要kubernetes之外的设备的支持。

Ingress相当于一个七层的负载均衡器，是kubernetes对反向代理的一个抽象，它的工作原理类似于Nginx，可以理解为Ingress里面建立了诸多映射规则，Ingress Controller通过监听这些配置规则并转化为Nginx的反向代理配置，然后对外提供服务。

![04](./ingress.png)

