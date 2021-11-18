- [**一、汇总：**](#一汇总)
  - [1、`docker`命令介绍](#1docker命令介绍)
- [**二、镜像相关**](#二镜像相关)
  - [1、拉取镜像](#1拉取镜像)
  - [2、查看镜像](#2查看镜像)
  - [3、推送镜像](#3推送镜像)
  - [4、删除镜像](#4删除镜像)
  - [5、创建镜像](#5创建镜像)
  - [6、搜索镜像](#6搜索镜像)
  - [7、登录远端镜像仓库](#7登录远端镜像仓库)
  - [8、查看镜像底层信息](#8查看镜像底层信息)
  - [9、镜像导入和导出](#9镜像导入和导出)
- [**三、容器相关**](#三容器相关)
  - [1、运行容器](#1运行容器)
  - [2、查看正在运行的容器](#2查看正在运行的容器)
  - [3、停止容器](#3停止容器)
  - [4、删除容器](#4删除容器)
  - [5、查看容器日志](#5查看容器日志)
  - [6、查看容器进程](#6查看容器进程)
  - [7、查看容器配置信息](#7查看容器配置信息)
  - [8、进入容器](#8进入容器)
  - [9、使用`docker cp`将文件从本地复制到容器](#9使用docker-cp将文件从本地复制到容器)
- [四、dockerfile](#四dockerfile)
  - [1、docker build](#1docker-build)
  - [2、常用命令](#2常用命令)
    - [FROM命令:](#from命令)
    - [ENV指令](#env指令)
    - [WORKDIR 指令：](#workdir-指令)
    - [RUN命令:](#run命令)
    - [EXPOSE指令：](#expose指令)
    - [CMD指令：](#cmd指令)
    - [**COPY** 命令:](#copy-命令)
    - [ADD命令：](#add命令)
    - [VOLUME 定义匿名卷:](#volume-定义匿名卷)
    - [USER 命令:](#user-命令)
  - [3、dockerfile用例](#3dockerfile用例)
- [引用](#引用)
## **一、汇总：**

* Docker环境信息 — `docker [info|version]`
* 容器生命周期管理 — `docker [create|exec|run|start|stop|restart|kill|rm|pause|unpause]`
* 容器操作运维 — `docker [ps|inspect|top|attach|wait|export|port|rename|stat]`
* 容器rootfs命令 — `docker [commit|cp|diff]`
* 镜像仓库 — `docker [login|pull|push|search]`
* 本地镜像管理 — `docker [build|images|rmi|tag|save|import|load]`
* 容器资源管理 — `docker [volume|network]`
* 系统日志信息 — `docker [events|history|logs]`

**常用命令的含义：**

### 1、`docker`命令介绍

```shell
docker --help
```

管理命令:
 `container`  管理容器
 `image`    管理镜像
 `network`   管理网络

命令：
 `attach`   介入到一个正在运行的容器
 `build`    根据`Dockerfile`构建一个镜像
 `commit`   根据容器的更改创建一个新的镜像
 `cp`     在本地文件系统与容器中复制 文件/文件夹
 `create`   创建一个新容器
 `exec`    在容器中执行一条命令
 `images`   列出镜像
 `kill`    杀死一个或多个正在运行的容器  
 `logs`    取得容器的日志
 `pause`    暂停一个或多个容器的所有进程
 `ps`     列出所有容器
 `pull`    拉取一个镜像或仓库到`registry`
 `push`    推送一个镜像或仓库到`registry`
 `rename`   重命名一个容器
 `restart`   重新启动一个或多个容器
 `rm`     删除一个或多个容器
 `rmi`     删除一个或多个镜像
 `run`     在一个新的容器中执行一条命令
 `search`   在`Docker Hub`中搜索镜像
 `start`    启动一个或多个已经停止运行的容器
 `stats`    显示一个容器的实时资源占用
 `stop`    停止一个或多个正在运行的容器
 `tag`     为镜像创建一个新的标签
 `top`     显示一个容器内的所有进程
 `unpause`   恢复一个或多个容器内所有被暂停的进程 

```shell
docker info  #查看系统(docker)层面信息，包括管理的images, containers数等
docker version #查看docker的版本号，包括客户端、服务端、依赖的Go等
```

## **二、镜像相关**

### 1、拉取镜像

```shell
# docker pull <image> 从docker registry server 中下拉image
docker pull nginx
```

### 2、查看镜像

```shell
docker images ##过滤掉中间镜像（现有镜像的父镜像）
docker images -a ##列出所有的images
```

### 3、推送镜像

```shell
docker push <image|repository> #推送一个image或repository到registry
docker push <image|repository>:TAG #同上，指定tag
```

### 4、删除镜像

```shell
docker rmi
```

常用参数：

`-f`：强制删除运行中的容器

### 5、创建镜像

 （1）对源镜像更改后重新建立新镜像

```shell
docker commit <container> [repo:tag] ##将一个container固化为一个新的image，后面的repo:tag可选
```

常用参数：

`-m：`本次提交信息

`--author=""` ：作者

（2）使用`Dockerfile`文件来构建镜像

```shell
docker build
```

常用参数：

1. `-t x/y:z`：指定镜像的命名空间为`x`，仓库为`y`，`tag`为`z`

### 6、搜索镜像

```shell
docker search nginx
```

### 7、登录远端镜像仓库

```shell
docker login --username=yourhubusername --email=youremail@company.com
```

### 8、查看镜像底层信息

```shell
docker inspect <image|container> ##查看image或container的底层信息
```

### 9、镜像导入和导出

```shell
##快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也更大。
docker save ##保存的是镜像（image），docker export 保存的是容器（container）；
docker load ##用来载入镜像包，docker import 用来载入容器包，但两者都会恢复为镜像；
docker load ##不能对载入的镜像重命名，而 docker import 可以为镜像指定新名称，如： docker import ubuntu.tar merge_gt/ubuntu:v1(新名称)。
```



## **三、容器相关**

### 1、运行容器

```shell
docker run
```

常用参数：

`--name`:为容器指定名称

`-it`:启动一个交互型容器，此参数为我们和容器提供了一个交互shell 

`-d`:创建后台型容器

`-restart=always`:容器退出后自动重启

`-restart=on-failure:x`:容器退出时如果返回值是非0，就会尝试重启x次

`-p x:y` :主机端口：容器端口

`-P`：随机分配一个49000到49900的端口

`-v`：创建数据卷

`-n` :指定dns

`-h` : 指定容器的hostname

`-e` ：设置环境变量

`-m` :设置容器使用内存最大值

`--net`: 指定容器的网络连接类型，支持`bridge/host/none/container`

`--link=x`: 添加链接到另一个容器x

`--expose=x`: 开放端口x

这里`docker create`和`docker run -it`创建的容器都是交互型容器

### 2、查看正在运行的容器

```shell
docker ps
```

常用参数：
`-a`：查看所有容器
`-l`:只列出最近创建的
`-n=x`:只列出最后创建的x个
`-q`: 只列出容器id

### 3、停止容器

```shell
docker stop  ##方式较温柔，慢慢的停止容器的运行
docker kill  ##方式简单粗暴，立即停止容器运行

docker start/stop/restart <container> ##开启/停止/重启container
docker start -i <container> ##启动一个container并进入交互模式
```

### 4、删除容器

```shell
docker rm <container...>  ##删除一个或多个container
docker rm `docker ps -a -q`  ##删除所有的container
docker ps -a -q | xargs docker rm  ##同上, 删除所有的container
```

常用参数：
`-f`：强制删除运行中的容器

### 5、查看容器日志

```shell
docker logs <container> ###查看container的日志，也就是执行命令的一些输出
```

常用参数：
`-f`：实时查看日志
`--tail=x`:查看最后x行
`-t`:查看日志产生的时间

### 6、查看容器进程

```shell
docker top
```

### 7、查看容器配置信息

```shell
docker inspect
```

常用参数：

```shell
-f='{{x}}'：查看x配置
```

### 8、进入容器

 （1）进入交互型容器

```shell
docker attch
```

（2）进入后台型容器

```shell
docker exec
```

常用参数：
`-it` 容器id `/bin/bash`：进入到后台容器



### 9、使用`docker cp`将文件从本地复制到容器

```shell
docker cp index.html hardcore_torvalds:usr/share/nginx/html/
```



## 四、dockerfile

### 1、docker build

```shell
docker build <path> ##寻找path路径下名为的Dockerfile的配置文件，使用此配置生成新的image
docker build -t repo[:tag] ##同上，可以指定repo和可选的tag
docker build - < <dockerfile> ###使用指定的dockerfile配置文件，docker以stdin方式获取内容，使用此配置生成新的image
```

### 2、常用命令

#### FROM命令:

既然我们是在原有的centos镜像的基础上做定制，那么我们的新镜像也一定是需要以centos这个镜像为基础的，而FROM命令则代表了这个意思，在DockerFile中，基础镜像是必须指定的，FROM指令的作用就是指定基础镜像，因此一个DockerFile中,FROM是必备的指令，而且就像java，python的import关键字一样，在DockerFile中，**FROM指令必须放在第一条指令的位置**

当然，这个时候可能有朋友会问了，我要是不想在其他的镜像上定制镜像怎么办呢，没问题啊，Docker 提供了scratch 这个虚拟镜像，如果你选择 FROM scratch 的话，则意味着你不以任何镜像为基础，接下来所写的指令将作为镜像的第一层开始存在，当然，在某些情况下，比如linux下静态编译的程序，运行的时候不需要操作系统提供运行时的支持，这个时候FROM scratch 是没有问题的，反而会大幅降低我们的镜像体积。

#### ENV指令

功能：**设置环境变量**

同样的，DockerFile也提供了两种格式：

- ENV  key  value
- ENV  key1=value1  key2=value2

这个指令很简单，就是设置环境变量而已，无论是后面的其它指令，如 RUN， 还是运行时的应用，都可以直接使用这里定义的环境变量。

可以看到我们示例中使用ENV设置mypath变量之后，在下一行WORKDIR则使用到了mypath这个变量

```shell
ENV mypath /tmp  ##设置环境变量
WORKDIR $mypath ###指定工作目录
```

#### WORKDIR 指令：

功能，**指定工作目录**

格式为：WORKDIR  工作目录路径，如果这个目录不存在的话，WORKDIR则会帮助我们创建这个目录。

设置过工作目录之后，当我们启动容器，会直接进入该工作目录

```shell
[root@8081304919c9 tmp]#
```

#### RUN命令:

**RUN 指令是用来执行命令行命令的**。由于命令行的强大能力，RUN 指令也是在定制镜像时是较为常用的指令之一。

RUN命令的格式一共有两种，分别是:

- `Shell`格式

    `RUN``命令，就像直接在命令行中输入命令一样，比如`RUN yum -y install vim`就是使用的这种格式

- `exec`格式

    RUN["可执行文件","参数1","参数2"]，感觉就像调用函数一样

就像我们在上一篇文章中说过的那样，DockerFile中每一条指令都会建立一层，比如我们上面执行过下面这条命令

```shell
RUN yum -y install vim
```

执行结束之后，则调用commit提交这一层的修改，使之构成一个新的镜像，怎么样，是不是豁然开朗了呢。

同样的，`Dockerfile`支持`Shell`类的行尾添加 `\`的命令换行方式，以 及行首`#`进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。

> 提示：
>
> 如果使用apt方式安装的话，最后不要忘记清理掉额外产生的apt缓存文件，如果不清理的话会让我们的镜像显得非常臃肿。因为DockerFile生成一层新的镜像的时候，并不会删除上一层镜像所残留的文件。

#### EXPOSE指令：

功能：**声明端口**

格式： EXPOSE   端口1   端口2

EXPOSE 指令是声明运行时容器提供服务端口，这当然只是一个声明，在运行时并不会因为这个声明应用就会开启这个端口的服务。这样声明主要是为了方便后期我们配置端口映射。

#### CMD指令：

之前介绍容器的时候曾经说过，Docker不是虚拟机，容器就是进程。既然是进程，那么在启动容器的时候，需要指定所运行的程序及参数。CMD指令就是用于指定默认的容器主进程的启动命令的。

同样的，DockerFile也为我们提供了两种格式来使用CMD命令:

- `shell`格式：`CMD`命令
- `exec` 格式：`CMD ["可执行文件", "参数 1", "参数 2"...]`

示例中，我们使用的是第一种：

```shell
CMD /bin/bash
```

这条指令带来的效果就是，**当我们通过run -it 启动命令的时候，容器会自动执行/bin/bash，centos默认也是CMD /bin/bash，所以当我们运行centos镜像的时候，会自动进入bash环境里面。**

当然，我们也可以通过运行时指定命令的方式来体换默认的命令，比如:

```shell
docker run -it centos cat /etc/os-release
```

这样当我们运行镜像的时候，`cat /etc/os-release`就会替代默认的`CMD /bin/bash`输出系统的版本信息了。

如果使用`shell`格式的话， 实际的命令会被包装为`sh -c`的参数的形式进行执行。

比如：

```shell
CMD echo $HOME
```

在实际执行中，会将其变更为

```shell
CMD [ "sh", "-c", "echo $HOME" ]
```

当然还有很多初学者特别容易犯的问题，就是去启动后台服务，比如:

```shell
CMD service nginx start
```

这样子去用，会发现容器运行了一会就自动退出了。

我们之前不止一次的提醒过，**容器不是虚拟机，容器就是进程**，容器内的应用都应该以前台运行，而不是像虚拟机，物理机那样去运行后台服务，容器就是为了主进程而存在的，主进程退出，容器就失去了存在的意义，从而退出，其它辅助进程不是它需要关心的东西。

怎么理解呢？想想偶像剧，容器是女主角，主进程是男主角

你走了，我也不活了（撕心裂肺大哭），大概就是这么个意思。

正如我们前面所提出的，实际上`CMD service nginx start`最终会被理解为：

```shell
CMD [ "sh", "-c", "service nginx start"]
```

在这里，我们主进程实际就是sh，当我们`service nginx start`执行完毕之后，那么sh自然就会退出了，主进程退出，容器自然就会相应的停止。争取的做法是直接执行nginx可执行文件，并且声明以前台的形式运行:

```shell
CMD ["nginx", "-g", "daemon off;"]
```

到这里，我们示例中所涉及到的命令已经讲完了，当然，这并不够，Docker中仍然有很多命令是我们使用比较频繁的，下面我们的部分作为补充，讲一下其他常用的DockerFile命令。

#### **COPY** 命令:

功能:**复制文件**

Docker依旧提供了两种格式供我们选择:

- COPY [--chown=:] <源路径>... <目标路径>
- COPY [--chown=:] ["<源路径 1>",... "<目标路径>"]

到这里大家其实会发现，Docker提供的两种格式其实都是差不多的用法，一种类似于命令行，一种则类似于函数调用。

第一种例如(将package.json拷贝到/usr/src/app/目录下):

```shell
COPY package.json /usr/src/app/
```

其次，目标路径 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径 ，工作目录可以用 WORKDIR 指令来指定，如果需要改变文件所属的用户或者用户组，可以加上--chown 选项。

> 需要注意的是，使用 COPY 指 令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。这 个特性对于镜像定制很有用。

#### ADD命令：

ADD命令可以理解为COPY命令的高级版，格式和用法与COPY几乎一致，ADD在COPY的基础上增加了一些功能，比如源路径可以是一个URL链接，当你这么用的时候，Docker会尝试着先将该URL代表的文件下载下来，然后复制到目标目录上去，其他的则是在COPY的基础上增加了解压缩之类的操作，码字码的手疼，需要了解的朋友可以去官网查看相关的文档，这里我就不延申了。

#### VOLUME 定义匿名卷:

在上一篇中，我们有讲容器卷这个概念，为了防止运行时用户忘记 将动态文件所保存目录挂载为卷，在 Dockerfile 中，我们可以事先指定某些 目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运 行，不会向容器存储层写入大量数据。

例如:

```
VOLUME /data
复制代码
```

运行时通过-v参数即可以覆盖默认的匿名卷设置。

#### USER 命令:

功能:**指定当前用户**

格式:**USER  用户名:用户组**

`USER`指令和`WORKDIR`相似，都是改变环境状态并影响以后的层。`WORKDIR` 是改变工作目录，`USER`则是改变之后层的执行`RUN`, `CMD`以及`ENTRYPOINT`这类命令的身份。当然，和`WORKDIR`一样，`USER`只是帮助你切换到指定用户。

当然这个大前提是，你的`User`用户是事先存在好的。

### 3、dockerfile用例

```shell
FROM centos  ##继承至centos
ENV mypath /tmp  ##设置环境变量
WORKDIR $mypath ##指定工作目录

RUN yum -y install vim ##执行yum命令安装vim
RUN yum -y install net-tools ###执行yum命令安装net-tools

EXPOSE 80 ###对外默认暴露的端口是80
CMD /bin/bash ###CMD 容器启动命令，在运行容器的时候会自动执行这行命令，比如当我们 docker run -it centos 的时候，就会直接进入bash


##然后编译该镜像
docker build -f ./DockerFile -t mycentos:1.3.
-t ##新镜像名字:版本
-f ###文件 
-d 文件夹
```



## 引用

1. [dockerfile的最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

2. [docker命令](https://docs.docker.com/engine/reference/commandline/cli/)

