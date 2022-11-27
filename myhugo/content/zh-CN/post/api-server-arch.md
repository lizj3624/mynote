---
title: "Kubernetes Api Server详解"
date: 2022-11-27T22:29:51+08:00
tags:
   - kubernetes
   - cloudnative
   - api-server 
categories:
   - kubernetes
   - cloudnative
   - api-server 
toc: true
---

## 介绍

整个`Kubernetes`技术体系由声明式API以及Controller构成，而`kube-apiserver`是`Kubernetes`的声明式`api server`，并为其它组件交互提供了桥梁。因此`kube-apiserve`是`Kubernetes`体系中非常重要的组件之一，加深对其理解就显得至关重要。

`kube-apiserver`的核心功能是提供了`Kubernetes`各类资源对象（如Pod、RC、Service等）的增、删、改、查及Watch等HTTP Rest接口，成为集群内各个功能模块之间数据交互和通信的中心枢纽，是整个系统的数据总线和数据中心。其主要运行在master节点上。

`kube-apiserver`的功能：

- 提供了集群管理的REST API接口(包括认证授权、数据校验以及集群状态变更)。
- 提供其他模块之间的数据交互和通信的枢纽（其他模块通过API Server查询或修改数据，只有API Server才直接操作etcd）。
- 是资源配额控制的入口。
- 拥有完备的集群安全机制。

## 架构

`kube-apiserver`从上到下可以分为四层：接口层，访问控制层，注册表层和数据库层。

![01](./apiserver-arch.png)

### API层

API层主要有三个Server组成

1. Kube Core API Server 核心接口服务器

负责对请求的一些通用处理，认证、鉴权等，以及处理各个内建资源的 REST 服务。

2. API Extensions Server 可扩展接口服务器

主要处理 CustomResourceDefinition（CRD）和 CustomResource（CR）的 REST 请求，也是 Delegation 的最后一环，如果对应 CR 不能被处理的话则会返回 404。

3. Aggregator Server 聚合服务器

暴露的功能类似于一个七层负载均衡，将来自用户的请求拦截转发给其他服务器，并且负责整个 APIServer 的 Discovery 功能。

### 访问控制层

访问控制层对用户进行身份鉴权，然后根据配置的各种访问许可逻辑（Admission Controller），判断是否允许访问。当请求到达 `kube-apiserver` 时，`kube-apiserver `首先会执行在 `http filter chain `中注册的过滤器链，该过滤器对其执行一系列过滤操作，主要有认证、鉴权等检查操作。

### 注册表层

Kubernetes 把所有资源都保存在注册表（Registry）中，针对注册表中的各种资源对象，都定义了相应的资源对象类型，以及如何创建资源，转换不同版本的资源，编码解码资源。

### Etcd层

通过KV持久化存储Kubernetes对象，API Server 就是利用Etcd的Watch特性实现了经典List-Watch机制。

## 与其他组件交互

`kube-apiserver`作为集群内各个功能模块之间数据交互和通信的中心枢纽，我们看一下与其他功能模块交互。

1. 与kubelet交互

每个Node节点上的kubelet定期就会调用`kube-apiserver`的REST接口报告自身状态，`kube-apiserver`接收这些信息后，将节点状态信息更新到etcd中。kubelet也通过`kube-apiserver`的watch接口监听Pod信息，从而对Node机器上的Pod进行管理。

2. 与kube-controller-manager交互

kube-controller-manager中的Node Controller模块通过`kube-apiserver`提供的watch接口，实时监控Node的信息，并做相应处理。

3. 与kube-scheduler交互

kube-scheduler通过`kube-apiserver`的Watch接口监听到新建Pod副本的信息后，它会检索所有符合该Pod要求的Node列表，开始执行Pod调度逻辑。调度成功后将Pod绑定到目标节点上。

如上组件都通过其REST接口交互，接口大致分为如下几类：

![02](./API-server-space-API.png)

## 访问

有多种方式可以访问`kube-apiserver`提供的 REST API：

1. [kubectl](https://kubernetes.io/zh-cn/docs/reference/kubectl/)命令行工具

2. 支持多种语言的SDK
- [Go](https://github.com/kubernetes/client-go)
- [Python](https://github.com/kubernetes-incubator/client-python)
- [Javascript](https://github.com/kubernetes-client/javascript)
- [Java](https://github.com/kubernetes-client/java)
- [CSharp](https://github.com/kubernetes-client/csharp)
- 其他 [OpenAPI](https://www.openapis.org/) 支持的语言，可以通过 [gen](https://github.com/kubernetes-client/gen) 工具生成相应开发语言的client
3. 直接调用REST API接口
   
   这个需要创建管理员用户，并授权以及获取token。
   
   ```bash
   # -k 允许curl使用非安全的ssl连接并且传输数据（证书不受信）
   $ curl -k --header "Authorization: Bearer $TOKEN"  https://192.168.0.113:6443/api
   ```

## 引用

1.  [一文读懂 Kubernetes APIServer 原理](https://segmentfault.com/a/1190000039032667)

2. [Kubernetes API | Kubernetes](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/)

3. [Kubernetes API Server 深入浅出](https://loulan.me/post/kube-apiserver/)

4. [Kubernetes 技术架构深度剖析](https://www.infvie.com/ops-notes/kubernetes-in-depth-analysis-of-technical-architecture.html)

5. [Kubernetes API Server handler 注册过程分析 | 云原生社区](https://cloudnative.to/blog/apiserver-handler-register/)

6. [Kubernetes 核心组件：API Server 概念/功能](https://blog.csdn.net/qq_34556414/article/details/125711133)
