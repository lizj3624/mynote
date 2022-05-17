---
title: "CentOS8 安装部署Kubernetes-1.20"
date: 2022-05-16T18:15:21+08:00
tags:
   - kubernetes
   - cloudnative
   - centos
categories:
   - kubernetes
   - cloudnative
   - centos
toc: true
---

`CentOS8`环境通过`kubeadm`工具安装部署`kubernetes`集群，集群有三台虚拟机组成，一台`master`，两个`node`。

## 环境准备

准备三台`CentOS8`服务器，主机名与静态IP地址如下表所示（参考下边 master 节点服务器的配置）：

| 角色     | 主机名    | ip地址           |
| ------ | ------ | -------------- |
| master | master | 192.168.56.104 |
| node   | node1  | 192.168.56.101 |
| node   | node2  | 192.168.56.108 |

```shell
[root@myk8s-01 ~]# cat /etc/redhat-release
CentOS Linux release 8.2.2004 (Core)
```

下面以 `master` 服务器为例，进行相应的配置（node 节点也需要做同样的配置）

### 查看系统版本

```shell
[root@myk8s-04 ~]# cat /etc/centos-release
CentOS Linux release 8.3.2011
```

### 关闭防火墙

```shell
[root@myk8s-04 ~]# systemctl stop firewalld
[root@myk8s-04 ~]# systemctl disable firewalld
```

### 配置网络

```shell
[root@myk8s-04 ~]# cat /etc/sysconfig/network-scripts/ifcfg-enp0s3
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=enp0s3
UUID=8583e7ca-8aa4-46b8-960e-7e055f8dd626
DEVICE=enp0s3
ONBOOT=yes
HWADDR=08:00:27:62:C1:C1
```

### 添加阿里源

```shell
[root@myk8s-04 ~]# rm -rfv /etc/yum.repos.d/*
[root@myk8s-04 ~]# curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
```

### 配置主机名

```shell
## 设置主机名
[root@myk8s-04 ~]# hostnamectl set-hostname myk8s-08.host.com

## 将如下加入/etc/hosts
[root@myk8s-04 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.56.104 myk8s-03.host.com master
192.168.56.108 myk8s-08.host.com node
192.168.56.101 myk8s-01.host.com node
```

### 关闭selinux

```shell
[root@myk8s-04 ~]# sed -i 's/enforcing/disabled/' /etc/selinux/config
[root@myk8s-04 ~]# setenforce 0

```

### 关闭swap，注释swap分区

```shell
[root@myk8s-04 ~]# swapoff -a

## 将/etc/fstab中最后一行注释掉 #/dev/mapper/cl-swap swap swap defaults 0 0
[root@myk8s-04 ~]# vim /etc/fstab
```

## 安装部署docker

CentOS8安装部署docker步骤请参考[这里](https://lizj3624.github.io/post/centos-docker/)

安装相应的依赖包

```shell
[root@myk8s-04 ~] yum install vim bash-completion net-tools gcc -y
```

### 添加aliyun docker仓库加速器

```shell
[root@myk8s-04 ~]# mkdir -p /etc/docker
[root@myk8s-04 ~]# cat /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "registry-mirrors": ["https://jxfzcj2d.mirror.aliyuncs.com"]
}


## 重新加载配置，重启docker
[root@myk8s-04 ~]# systemctl daemon-reload
[root@myk8s-04 ~]# systemctl restart docker
```

> 阿里云docker镜像加速器文档参考[这里](https://developer.aliyun.com/article/29941)

### 测试验证

```shell
[root@myk8s-04 ~]# docker info

## 保证Cgroup Driver是systemd
[root@myk8s-04 ~]# docker info|grep "Cgroup Driver"
 Cgroup Driver: systemd
```

> 1. 安装时遇到问题，CentOS8默认安装了`Podman`，再次安装`docker`时提升冲突，因此需要协助`Podman`和`Buildah`，再次安装`docker`
> 
> 2. Cgroup Driver不是systemd的时候，改/etc/docker/daemon.json中的"exec-opts": ["native.cgroupdriver=systemd"]



## 安装kubectl、kubelet、kubeadm

```shell
# 1. 添加阿里kubernetes源
[root@myk8s-04 ~]# cat /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg

# 2. 安装部署
[root@myk8s-04 ~]# yum -y install kubectl kubelet kubeadm
[root@myk8s-04 ~]# systemctl enable kubelet
```

## 初始化k8s集群

### kubeadm初始化脚步

```shell
[root@myk8s-04 ~]# kubeadm init --kubernetes-version=1.21.1  \
> --apiserver-advertise-address=192.168.56.104   \
> --image-repository registry.aliyuncs.com/google_containers  \
> --service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
```

> 由于kubeadm 默认从官网k8s.grc.io下载所需镜像，国内无法访问，因此需要通过–image-repository指定阿里云镜像仓库地址。



POD的网段为: `10.122.0.0/16`， `APIServer`地址就是`master`本机IP。

集群初始化成功后返回如下信息：

```shell
[init] Using Kubernetes version: v1.21.1
[preflight] Running pre-flight checks
        [WARNING FileExisting-tc]: tc not found in system path
        [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.1. Latest validated version: 19.03
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local master] and IPs [10.10.0.1 192.168.1.25]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [localhost master] and IPs [192.168.1.25 127.0.0.1 ::1]
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [localhost master] and IPs [192.168.1.25 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Starting the kubelet
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[kubelet-check] Initial timeout of 40s passed.
[apiclient] All control plane components are healthy after 98.007042 seconds
[upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.20" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --upload-certs
[mark-control-plane] Marking the node master as control-plane by adding the labels "node-role.kubernetes.io/master=''" and "node-role.kubernetes.io/control-plane='' (deprecated)"
[mark-control-plane] Marking the node master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: mwxojd.djyh86ktwwwyv0qp
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to get nodes
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.56.104:6443 --token n9ymj8.nscmc7jz344d9maa \
 --discovery-token-ca-cert-hash sha256:13164b268a07c60fcfe9b71cfba98dc36f630a7c64a471397fd52cf3631f46a7
```

> 记录生成的最后部分内容(kubeadm内容)，此内容是在其它节点加入Kubernetes集群时执行。

### 创建`kubectl`

```shell
[root@myk8s-04 ~]# mkdir -p $HOME/.kube
[root@myk8s-04 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@myk8s-04 ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
[root@myk8s-04 ~]# source <(kubectl completion bash) && echo 'source <(kubectl completion bash)' >> ~/.bashrc
```

### 查看节点pod

```shell
[root@myk8s-04 ~]# kubectl get node
NAME                STATUS   ROLES                  AGE   VERSION
myk8s-04.host.com   Ready    control-plane,master   28h   v1.21.1

[root@master ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                             READY   STATUS    RESTARTS   AGE
kube-system   coredns-7f89b7bc75-6jzcr         0/1     Pending   0          11m
kube-system   coredns-7f89b7bc75-fzq95         0/1     Pending   0          11m
kube-system   etcd-master                      1/1     Running   0          11m
kube-system   kube-apiserver-master            1/1     Running   0          11m
kube-system   kube-controller-manager-master   1/1     Running   0          11m
kube-system   kube-proxy-t8jlr                 1/1     Running   0          11m
kube-system   kube-scheduler-master            1/1     Running   0          11m

```

> master当时Ready状态就说明成功了

## 安装calico网络

### 安装Calico网络

```shell
[root@myk8s-04 ~]# kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
poddisruptionbudget.policy/calico-kube-controllers created
```

### 查看pod和node

```shell
[root@myk8s-04 ~]# kubectl get pod --all-namespaces
NAMESPACE     NAME                                       READY   STATUS              RESTARTS   AGE
kube-system   calico-kube-controllers-744cfdf676-87n8d   0/1     ContainerCreating   0          45s
kube-system   calico-node-kfnf5                          0/1     PodInitializing     0          46s
kube-system   coredns-7f89b7bc75-6jzcr                   0/1     ContainerCreating   0          13m
kube-system   coredns-7f89b7bc75-fzq95                   0/1     ContainerCreating   0          13m
kube-system   etcd-master                                1/1     Running             0          13m
kube-system   kube-apiserver-master                      1/1     Running             0          13m
kube-system   kube-controller-manager-master             1/1     Running             0          13m
kube-system   kube-proxy-t8jlr                           1/1     Running             0          13m
kube-system   kube-scheduler-master                      1/1     Running             0          13m

[root@myk8s-04 ~]# kubectl get node
NAME     STATUS   ROLES                  AGE   VERSION
master   Ready    control-plane,master   13m   v1.20.1

```

## 向集群中加入node

`node`节点也需要执行如上的`环境准备`、`安装docker`、`安装kubectl kubeadm kubelet`的步骤，然后再执行如下步骤

```
# 1. 更新admin.conf文件，每个node节点都比跟master的保持一致
[root@myk8s-04 ~]# scp /etc/kubernetes/admin.conf root@192.168.56.101:/etc/kubernetes

# 2. 在节点1执行如下命令
```shell
[root@myk8s-01 ~]# mkdir -p $HOME/.kube
[root@myk8s-01 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@myk8s-01 ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/confi

[root@myk8s-01 ~]# echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
[root@myk8s-01 ~]# source ~/.bash_profile

# 3. 加入node节点到集群，也就master初始化成功后返回在master上查看节点信息
[root@myk8s-01 ~]# kubeadm join 192.168.56.104:6443 --token n9ymj8.nscmc7jz344d9maa \
 --discovery-token-ca-cert-hash sha256:13164b268a07c60fcfe9b71cfba98dc36f630a7c64a471397fd52cf3631f46a7


# 4. 查看节点
[root@myk8s-01 ~]# kubectl get nodes
NAME                STATUS   ROLES                  AGE     VERSION
myk8s-01.host.com   Ready    <none>                 5h14m   v1.21.1
myk8s-04.host.com   Ready    control-plane,master   5h27m   v1.21.1
myk8s-08.host.com   Ready    <none>                 113m    v1.21.1
```

> node 节点的 docker 配置文件 /etc/docker/daemon.json 尽量跟 master 节点保持一致   
> 可以通过这个命令：`journalctl -f -u kubelet` 
> 引用：[k8s 集群之使用 kubeadm 在 Centos8 上部署 kubernetes 1.20](https://blog.csdn.net/qq_34596292/article/details/112131042)
