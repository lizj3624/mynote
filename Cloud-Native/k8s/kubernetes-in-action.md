# Pod：运行于kubernetes中的容器
Pod是一组并置的容器，是kubernetes调度和管理的最小单位
1. 一个Pod的所有容器都运行在同一个节点上，绝不运行跨节点。
2. 同一个Pod的容器都在相同的network和UTS命名空间下，他们共享相同的主机名和网络接口，也在相同的IPC命令空间下。由于同一Pod下共享网络接口，就拥有相同的IP和端口空间，因此不同进程不能监听相同端口。
3. kubernetes集群中的所有Pod都在同一个共享网络下。
4. 一个Pod可以放在相关联的服务容器，
5. 通过YAML和JSON创建Pod
   * metadata 包括名称、命名空间、标签和关于该容器 的其他信息 
   * spec 包含 pod 内容的实际说明 ， 例如 pod 的容器、卷和其他数据 
   * status包含运行中的pod的当前信息，例如pod所处的条件、 每个容器的描述和状态，以及内部 IP 和其他基本信息 。

```shell
# 根据YAML创建Pod资源
kubectl create -f kubia-manual.yaml
kubectl apply -f kubia-manual.yaml

# 获取Pod信息
kubectl get po kubia-manual -o yaml

# 查看所有Pod
kubectl get pods

# 查看日志
kubectl logs kubia-manual
```

6. 使用标签组织Pod
```shell
kubectl get po --show-labels

kubectl get po -L creation_method,env
```

7. 注解

8. 命名空间

9. 停止和移除
```shell
# 按名称删除 kubia-gpu pod
kubectl delete po kubia-gpu
```
