# 基础入门
## docker核心概念
1. 容器: Container
2. 镜像: Image
3. 仓库: Repository

## 使用docker镜像
### 获取镜像
```shell
# docker [image] pull NAME:[TAG]
# NAME是镜像的名字，TAG是标签

docker pull ubuntu:18.04

docker pull registry.hub.docker.com/ubuntu:18.04
```

### 查看镜像信息
```shell
docker image  

docker tag ubuntu:latest myybuntu:latest

docker inspect ubuntu:18.04

docker history ubuntu:18.04
```

### 查找
```shell
docker search --filter=stars=4 nginx
```

### 删除
```shell
docker rmi myubuntu:latest
```

### 创建容器
```shell
docker commit -m "add new file" -a "Docker Newbee" a925cb40b3f0 test:0.1

cat ubuntu-18.04-x86_64-minimal.tar.gz |docker import - ubuntu:18.04

# docker file
```

### 存出和载入镜像
```shell
docker load -i ubuntu_18.04.tar

docker load < ubuntu_18.04.tar


docker tag test:latest user/test:latest
docker push user/test:latest
```

## 操作容器
```shell
## 创建容器
docker create -it ubuntu:latest

## 启动已经创建的容器
docker start af

## 创建并启动容器
docker run ubuntu /bin/echo 'Hello workd'

-d 守护

## 停止
docker pause [contains]

docker stop ce5

docker restart ce5

## 进入容器
docker attach 

docker exec -it 243c32535da7 /bin/bash

## 删除
docker rm ce554267d7a4
```
## docker数据管理
数据卷将主机操作系统的目录直接映射到容器，类型Linux的mount行为
```shell
docker volume create -d local test

# -mount 选项支持三种类型的数据卷，包括 :
# volume : 普通数据卷，映射到主机/ var/lib/docker/volumes 路径下;
# bind:绑定数据卷，映射到主机指定路径下;
# tmpfs :临时数据卷，只存在于内存中 。
docker run d P -name web mount type=bind,source=/webapp,destination=/opt/ webapp training/webapp python app.py

docker run -d -P --name web -v /webapp:/opt/webapp training/webapp python app.py

# 只读 ro
docker run -d -P --name web -v /webapp: /opt/webapp:ro training/webapp python app.py

# 数据卷容器
docker run -it -v /dbdata --name dbdata ubuntu

docker run -it --volumes-from dbdata -name db1 ubuntu

# 数据卷容器备份和恢复
docker run -volumes-from dbdata -v $ (pwd) :/backup - -name worker ubuntu tar
cvf /backup/backup.tar /dbdata

docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
```

## 端口映射与容器互联
当容器中运行一些网络应用， 要让外部访问这些应用时， 可以通过`-P`或`-p`参数来指 定端口映射。当使用平(大写的)标记时， Docker会随机映射一个`49000~49900`的端口 到内部容器开放的网络端口:
```shell
# 本地主机的49155被映射到了容器的5000端口
docker run -d -P training/webapp python app.py

# -p (小写的)则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。 支持的格式有 IP:HostPort:ContainerPort | IP::ContainerPort | HostPort:ContainerPort。

docker run -d -p 5000:5000 training/webapp python app.py

docker run -d -p 5000:5000 -p 3000:80 training/webapp py thon app.py

docker run -d -p 127.0.0.1::5000 training/webapp python app.py

docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py

# 查看端口映射
docker port nostalgic_morse 5000

docker run -d -P --name web --link db:db 扛aining/webapp py七hon app.py
```
## Dockerfile创建镜像
```shell
docker file
```

# 进阶
## 核心技术
### 基础架构
1. 服务端
   * dockerd：为客户端提供RESTful API，响应来自客户端的请求，采用模块化的架构，通过专门的Engine模块来分发管理各个来自客户端的任务。
   * docker-proxy：是dockerd的子进程，当需要进行容器端口映射时，docker-proxy完成网络映射配置
   * containerd：是dockerd的子进程，提供gRPC接口响应来自dockerd的请求，对下管理runC镜像和容器环境。
   * containerd-shim：是containerd的子进程，为runC容器提供支持，同时作为容器内进程的根进程
> runC是从docker公司开源的libcontainer项目演化而来的，目前加入OCI(Open Containers Initiative)，支持容器相关的技术栈，同时正在实现跨OS    

![arch](https://github.com/lizj3624/mynote/blob/master/Cloud-Native/pictures/docker-arch.jpg)
2. 客户端
docker命令就是客户端

3. 镜像仓库
docker hub

### 命名空间
命名空间(namespace)是Linux内核的一个强大特性，为容器虚拟化的实现提供极大便利，每个容器都可以拥有自己单独的命名空间。
实现了内存、CPU、网络IO、硬盘IO、存储空间，还有文件系统、网络、PID、UID、IPC等相互隔离
1. 进程命名空间
2. IPC命名空间
3. 网络命名空间
4. 挂载命名空间
5. UTS命名空间
6. 用户命名空间

### 控制组
控制组(CGroups)是Linux内核的一个特性
1. 资源限制
2. 优先级
3. 资源审计
4. 隔离
5. 控制

### 联合文件系统
联合文件系统(UnionFS)是一种轻量级的高性能分层文件系统，它支持将文件系统中的修改信息作为一次提交，并层层叠加，同时可以将不同目录挂载到同一个虚拟文件系统下，应用看到的挂载的最终结果。是docker镜像的技术基础

### Linux网络虚拟化
docker中网络接口默认是虚拟接口.docker服务启动时首先在主机上自动创建一个docker0虚拟网桥，实际上是一个Linux网桥。网桥可以理解为一个软件交换机，负责挂载其上的接口之间进行包转发。同时，Docker随机分配一个本地未占用的私有网段中的一个地址给docker0接口，比如`172.17.0.0、16`，掩码为`255.255.0.0.`，此后启动的容器的网口也会自动分配一个该网段的地址。当创建一个Docker容器的时候，同时会创建了一对`veth pair`互联接口。当向任一个接口发送包时，另外一个接口自动收到相同的包。互联接口的一端位于容器内，即`eth0`；另一端在本地并被挂载到`docker0`网桥，名称以`veth`开头。通过这种方式，主机可以与容器通信，容器之间也可以相互通信。如此一来，Docker就创建了在主机和所有容器之间一个虚拟共享网络。    

![arch](https://github.com/lizj3624/mynote/blob/master/Cloud-Native/pictures/docker-net.jpg)