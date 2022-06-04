---
title: "kubernetes中配置管理ConfigMap"
date: 2022-06-04T09:45:55+08:00
tags:
   - kubernetes
   - cloudnative
   - configMap 
categories:
   - kubernetes
   - cloudnative
   - configMap 
toc: true
---

### CongfigMap
ConfigMap对象用于为容器中的应用提供配置文件等信息，它们将相应的配置信息保存于对象中，而后在Pod资源上以存储卷的形式挂载并获取相关的配置，以实现配置与镜像文件的解耦。
例如为Tomcat的JVM配置堆内存大小等，在容器中启动时，我们可以向容器命令传递参数，将定义好的配置文件嵌入镜像文件中、通过环境变量(Environment Variables)传递配置数据，以及基于Docker卷传送配置文件等。

### 创建ConfigMap
ConfigMap的配置样例`myconfig.yaml`
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: game-demo
data:
  # 类属性键；每一个键都映射到一个简单的值
  player_initial_lives: "3"
  ui_properties_file_name: "user-interface.properties"

  # 类文件键
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5    
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
```

通过命令创建`kubectl create myconfig.yaml`


### Pod中使用ConfigMap
可以使用四种方式来使用`ConfigMap`配置`Pod`中的容器：
1. 在容器命令和参数内
2. 容器的环境变量
3. 在只读卷里面添加一个文件，让应用来读取
4. 编写代码在`Pod`中运行，使用`Kubernetes API`来读取`ConfigMap`

下面是一个`Pod`的示例，它通过使用`game-demo`中的值来配置一个`Pod`：
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: configmap-demo-pod
spec:
  containers:
    - name: demo
      image: alpine
      command: ["sleep", "3600"]
      env:
        # 定义环境变量
        - name: PLAYER_INITIAL_LIVES # 请注意这里和 ConfigMap 中的键名是不一样的
          valueFrom:
            configMapKeyRef:
              name: game-demo           # 这个值来自 ConfigMap
              key: player_initial_lives # 需要取值的键
        - name: UI_PROPERTIES_FILE_NAME
          valueFrom:
            configMapKeyRef:
              name: game-demo
              key: ui_properties_file_name
      volumeMounts:
      - name: config
        mountPath: "/config"
        readOnly: true
  volumes:
    # 你可以在 Pod 级别设置卷，然后将其挂载到 Pod 内的容器中
    - name: config
      configMap:
        # 提供你想要挂载的 ConfigMap 的名字
        name: game-demo
        # 来自 ConfigMap 的一组键，将被创建为文件
        items:
        - key: "game.properties"
          path: "game.properties"
        - key: "user-interface.properties"
          path: "user-interface.properties"
```

### 引用
1. [ConfigMap](https://kubernetes.io/zh/docs/concepts/configuration/configmap/)

2. [Kubernetes之ConfigMap详解及实践](https://zhuanlan.zhihu.com/p/185988609)
