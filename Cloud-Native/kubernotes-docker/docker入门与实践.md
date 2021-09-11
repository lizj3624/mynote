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