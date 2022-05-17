---
title: "Kubernetes集群重置"
date: 2022-05-17T19:32:41+08:00
tags:
   - cloudnative
   - kubernetes
categories:
   - cloudnative
   - kubernetes
toc: true
---

1. 移除所有工作节点
```shell
## 查看节点
kubectl get node

## 删除指定节点
kubectl delete node <node-name> 
```

2. 所有工作节点删除工作目录，并重置kubeadm
```
rm -rf /etc/kubernetes/*
kubeadm reset
```

3. Master节点删除工作目录，并重置kubeadm
```shell
rm -rf /etc/kubernetes/*

rm -rf ~/.kube/*

rm -rf /var/lib/etcd/*

rm -rf /var/lib/cni/

rm -fr /etc/cni/net.d

## 重置
kubeadm reset -f
```

4. 重新init kubernetes
```shell
kubeadm init 
```
