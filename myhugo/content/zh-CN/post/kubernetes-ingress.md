---
title: "Kubernetes中的Ingress资源"
date: 2022-05-21T18:06:45+08:00
tags:
   - cloudnative
   - kubernetes
   - ingress 
categories:
   - cloudnative
   - kubernetes
   - ingress
toc: true
---
`Ingress`是为`kubernetes`集群中服务`service`提供对外访问的接口，主要`HTTP/HTTPS`访问。
`Ingress`公开了从集群外部到集群内服务的`HTTP`和`HTTPS`路由。流量路由由`Ingress`资源上定义的规则控制。

下面是一个将所有流量都发送到同一`Service`的简单`Ingress`示例：
![01](./ingress.png)

`Ingress`是由`Ingress controller`、`Ingress`资源和相应的`service`资源。最常见的`Ingress`控制器是
`kuberneteres`官方开源的[ingress-nginx](https://github.com/kubernetes/ingress-nginx/blob/main/README.md#readme)，
还有其他版本的控制器，在`kubernetes`官方登记的[其他控制器](https://kubernetes.io/zh/docs/concepts/services-networking/ingress-controllers/)。
主要根据`ingress-nginx`介绍一下`ingress`。

### 创建控制器
`ingress-nginx`已经写好创建控制器的`yaml`文件，可以直接创建，有特殊需求的可以根据这个文件改进一下。
```shell
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml

## 查看创建controller
$ kubectl get pods -n ingress-nginx
NAME                                        READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-6979d75b9d-2hf8f   1/1     Running   0          5h28m
```
> 具体ingress-nginx创建可以查看[这里](https://kubernetes.github.io/ingress-nginx/deploy/)

用如下步骤演示一下`ingress`的用法
### 创建ingress
`Ingress`资源需要指定`apiVersion`、`kind`、`metadata`和`spec`字段。
`spec`部分指明了域名(host)，url以及转发到`service`信息。

查看下面创建简单`Ingress`的yaml文件
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tomcat-ingress   # ingress的名字
  namespace: tomcat      # ingress所在命名空间
spec:
  rules:
    - host: tomcat.ns.com      # 域名
      http:
        paths:
          - path: /            #url
            pathType: Prefix
            backend:
              service:
                name: mytomcat-http  # service的名称
                port:
                  number: 8080       # service的端口
```

可以通过如下命令查看创建的`ingress`资源
```shell
## 创建
$ kubectl apply -f tomcat-app.yaml

## 查看
$ kubectl get ingress
```

### 创建service
部署应用服务(tomcat)，并创建服务(service)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcatinfra  ## 部署应用名称
  namespace: tomcat
spec:
  replicas: 1
  selector:
    matchLabels:
      app: tomcatinfra  ## 应用服务的标签，service通过这个标签选择应用的pods
  template:
    metadata:
      name: tomcatinfra
      labels:
        app: tomcatinfra
    spec:
      containers:
      - image: saravak/tomcat8   ## 指定应用的镜像
        name: tomcatapp
        ports:
         - containerPort: 8080

---
apiVersion: v1
kind: Service
metadata:
  name: mytomcat-http  ## 服务的名称
  namespace: tomcat    ## 命名空间，必要有Ingress, 应用在同一命名空间
spec:
  type: ClusterIP
  ports:
  - port: 8080         ## 服务端口
    targetPort: 8080   ## app pod端口
  selector:
    app: tomcatinfra   ## 应用名称
```

创建服务
```shell
## 创建
$ kubectl apply -f tomcat-srv.yaml

## 查看服务
$ kubectl get svc
```

这时候`nginx controller`中的nginx的配置应该新增了转发`tomcat.ns.com`的信息。
```shell
kubectl -n ingress-nginx exec -it nginx-ingress-controller-6979d75b9d-2hf8f -- cat nginx.conf
```
这时可以将域名`tomcat.ns.com`解析的IP改成`controller`所在宿主机的IP，然后`curl http://tomcat.ns.com/`将
请求转发到部署的`tomcat`应用上。

### 引用
1. [Ingress](https://kubernetes.io/zh/docs/concepts/services-networking/ingress/)

2. [Ingress 控制器](https://kubernetes.io/zh/docs/concepts/services-networking/ingress-controllers/)

3. [ingress nginx](https://kubernetes.github.io/ingress-nginx/)
