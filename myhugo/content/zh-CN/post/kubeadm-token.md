---
title: "Kubeadm中的Token过期问题"
date: 2022-05-22T10:04:56+08:00
tags:
   - kubernetes
   - cloudnative
   - kubeadm 
categories:
   - kubernetes
   - cloudnative
   - kubeadm 
toc: true
---

### kubeadm以及token
node节点加入`kubernetes`集群是通过在node节点执行`kubeadm join`命令完成的，具体类似如下：
```shell
$ kubeadm join 192.168.56.104:6443 --token l18cgl.jlp49w8zyz94vogn \
        --discovery-token-ca-cert-hash sha256:1672cd600e7e686b554ce79ee31a79d8fc8da34c339e7763d4601a149be8faa2
```
如上命令一般是`kubernetes`集群的master节点初始化成功后，返回的node节点加入的命令。
在参看[CentOS8 安装部署Kubernetes-1.20](https://lizj3624.github.io/post/centos8-kubernetes/)中介绍。
其中`token`是为加入node节点集群的标识，如果`token`不对或者过期时，node节点就加入失败。
`token`默认下是24小时，如果过期时，执行`kubeadm join`就是提示如下错误信息：
```shell
error execution phase preflight: couldn't validate the identity of the API Server: could not find a JWS signature in the cluster-info ConfigMap for token ID "jrc6he"
To see the stack trace of this error execute with --v=5 or higher
```

### token过期解决思路
master节点重新`token`，用新`token`再执行`kubeadm join`命令。

```shell
$ kubeadm token create  //默认有效期24小时,若想久一些可以结合--ttl参数,设为0则用不过期
jrc6he.dg2to0m4dhgrecv2

## 用新token，在执行kubeadm join

## 查看当前有效的token
$ kubeadm token list
```
