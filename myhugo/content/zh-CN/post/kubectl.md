---
title: "Kubectl常用命令"
date: 2022-05-14T14:53:41+08:00
tags:
   - kubernetes
   - cloudnative
   - kubtctl
categories:
   - kubernetes
   - cloudnative
   - kubtctl
toc: true
---

`kubectl`命令行是管理`kubernetes`集群的工具，学习好`kubectl`命令行对管理`kubernetes`很重要，一般安装`kubernetes`后都会自带这个工具。



## 语法

```shell
kubectl [command] [TYPE] [NAME] [flags]
```

* `command`：指定要对一个或多个资源执行的操作，例如 `create`、`get`、`describe`、`delete`。

* `TYPE`：指定[资源类型](https://kubernetes.io/zh/docs/reference/kubectl/overview/#%E8%B5%84%E6%BA%90%E7%B1%BB%E5%9E%8B)。资源类型不区分大小写， 可以指定单数、复数或缩写形式。例如`pod`、`pods`、`srv`等资源

* `NAME`：指定资源的名称。名称区分大小写。 如果省略名称，则显示所有资源的详细信息 `kubectl get pods`。

* `flags`: 指定可选的参数。例如，可以使用 `-s` 或 `-server` 参数指定 Kubernetes API 服务器的地址和端口。

```shell
kubectl get pods -A -o wide
```



## 常用命令

### 创建和更新资源

```shell
# 使用 example-service.yaml 中的定义创建服务。
kubectl apply -f example-service.yaml

# 使用 <directory> 路径下的任意 .yaml, .yml, 或 .json 文件 创建对象。
kubectl apply -f <directory>

# 使用 example-service.yaml 中的定义创建服务。
kubectl create -f example-service.yaml
```

> apply可以在资源不存在时创建，存在时根据配置重新修改资源，但是create只能在没有时创建，存在时会抛出错误



### 获取资源

```shell
# 以纯文本输出格式列出所有pod，并包含附加信息(如节点名)。
kubectl get pods -o wide

# 获取命名空间为ingress-nginx上pods，并以纯文本输出
kubectl get pods -n ingress-nginx -o wide

# 获取命名空间ingress-nginx中名字为nginx-ingress-controller-6979d75b9d-cbml4的pod的信息，并以文本格式输出
# 如果没有指明命名空间就是default命名空间
kubectl get pod -n ingress-nginx nginx-ingress-controller-6979d75b9d-cbml4 -o wide

# 查看所有命名空间
kubectl get namespaces

# 列出名字为web的rc
kubectl get replicationcontroller web

# 列表service信息
 kubectl get svc -o wide

# 获取所有resource
kubectl get all
```

* `kubectl describe` - 显示一个或多个资源的详细状态，默认情况下包括未初始化的资源。

```shell
# 查看所有节点描述信息
kubectl describe node

# 查看所有service描述信息
kubectl describe svc

# 查看名为nginx的service描述信息
kubectl describe svc nginx
```



### 删除资源

```shell
# 删除名称为nginx-6799fc88d8-chjq8的pod，这样删除后由于通过deployment或者rs创建的pod，新的pod还有可能被创建
kubectl delete pod nginx-6799fc88d8-chjq8

# 删除所有带有 '<label-key>=<label-value>' 标签的 Pod 和服务。
kubectl delete pods,services -l <label-key>=<label-value>

# 删除所有 pod，包括未初始化的 pod。
kubectl delete pods --all
```



### Pod容器操作

```shell
# 在pod的容器中执行ls命令
kubectl -n ingress-nginx exec -it nginx-ingress-controller-6979d75b9d-cbml4 -- /bin/ls

# 默认在pod 123456-7890的第一个容器中运行“date”并获取输出
$ kubectl exec 123456-7890 date

# 在pod 123456-7890的容器ruby-container中运行“date”并获取输出
$ kubectl exec 123456-7890 -c ruby-container date

# 切换到终端模式，将控制台输入发送到pod 123456-7890的ruby-container的“bash”命令，并将其输出到控制台/
# 错误控制台的信息发送回客户端。
$ kubectl exec 123456-7890 -c ruby-container -i -t -- bash -il
```



### Pod容器日志

```shell
# 返回仅包含一个容器的pod nginx的日志快照
kubectl logs nginx

# 返回pod ruby中已经停止的容器web-1的日志快照
kubectl logs -p -c ruby web-1

# 持续输出pod ruby中的容器web-1的日志
kubectl logs -f -c ruby web-1

# 仅输出pod nginx中最近的20条日志
kubectl logs --tail=20 nginx

# 输出pod nginx中最近一小时内产生的所有日志
kubectl logs --since=1h nginx

kubectl logs -f -l app=nginx --all-containers=true
```



### 其他

* `lable`标签

```shell
# 给名为foo的Pod添加label unhealthy=true
kubectl label pods foo unhealthy=true

# 给名为foo的Pod修改label为 'status' / value 'unhealthy'，且覆盖现有的value
kubectl label --overwrite pods foo status=unhealthy

# 给 namespace中的所有pod添加label
kubectl label pods --all status=unhealthy

# 仅当resource-version=1时才更新名为foo的Pod上的label
kubectl label pods foo status=unhealthy --resource-version=1
```

* `edit`编辑配置文件

```shell
  # 编辑名为“docker-registry”的service
  kubectl edit svc/docker-registry

  # 使用一个不同的编辑器
  KUBE_EDITOR="nano" kubectl edit svc/docker-registry

  # 编辑名为“docker-registry”的service，使用JSON格式、v1 API版本
  kubectl edit svc/docker-registry --output-version=v1 -o json
```

* cp命令

```shell
# 将“/tmp/foo_dir”本地目录拷贝到默认命名空间的远端pod的“/tmp/bar_dir”目录下
kubectl cp /tmp/foo_dir <some-pod>:/tmp/bar_dir

# 复制/tmp/foo本地文件到/tmp/bar在远程pod在一个特定的容器
kubectl cp /tmp/foo <some-pod>:/tmp/bar -c <specific-container>

# 将/tmp/foo文件拷贝到远程pod中的/tmp/bar目录下
kubectl cp /tmp/foo <some-namespace>/<some-pod>:/tmp/bar
```



## 引用

* [kubectl 概述 | Kubernetes](https://kubernetes.io/zh/docs/reference/kubectl/overview/)
