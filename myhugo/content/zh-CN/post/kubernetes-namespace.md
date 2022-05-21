---
title: "Kubernetes中的Namespace详解"
date: 2022-05-21T11:42:55+08:00
tags:
   - kubernetes
   - cloudnative
   - namespace 
categories:
   - kubernetes
   - cloudnative
   - namespace 
toc: true
---
在Kubernetes中，名字空间(Namespace)提供一种机制，将同一集群中的资源划分为相互隔离的组。
同一名字空间内的资源名称要唯一，但跨名字空间时没有这个要求。
名字空间作用域仅针对带有名字空间的对象例如`Deployment、Service`等，
这种作用域对集群访问的对象不适用例如`StorageClass、Node、PersistentVolume`等。

### 命名空间作用
命名空间为集群中的对象名称赋予作用域，命名空间还可以让用户轻松地将策略应用到集群的具体部分，
命名空间最大的好处之一是能够利用`Kubernetes RBAC`(基于角色的访问控制)。

1. 将命名空间映射到团队或项目上，为每个单独的项目或者团队创建一个命名空间。

2. 使用命名空间对生命周期环境进行分区，命名空间非常适合在集群中划分开发、staging以及生产环境。

3. 使用命名空间隔离不同的使用者，可以解决的用例是根据使用者对工作负载进行分段。

### 预配置的三个命名空间

1. `default`向集群中添加对象而不提供命名空间，这样它会被放入默认的命名空间中。

2. `kube-public`是让所有具有或不具有身份验证的用户都能全局可读。

3. `kube-system`用于Kubernetes管理的Kubernetes组件，一般规则是避免向该命名空间添加普通的工作负载。
> 三个预制的命名空间，有kubernetes创建和管理

### 使用命名空间
命名空间的使用可以有`kubectl`和`yaml`资源管理维护。

```shell
## 创建命名空间tomcat
kubectl create namespace tomcat 

## 查看命名空间
kubectl get ns

kubectl describe namespace tomcat 

## 查看命名空间tomcat上的pods, 通过-n参数指明命名空间
kubectl get pods -n tomcat

## 在命名空间上创建资源(service, ingress, pod)等，通过在资源文件指明namespace
namespace: tomcat

## 在指定命名空间运行pod 
kubectl run nginx --image=nginx --namespace=nginx

## 删除命名空间
kubectl delete namespace tomcat
```
还可以通过`yaml`的资源创建命名空间，然后执行`kubectl apply -f tomcat.yaml`
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: tomcat
```

### 引用
1. [命名空间](https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/namespaces/) 

2. [超长干货 | Kubernetes命名空间详解](https://segmentfault.com/a/1190000018199756)
