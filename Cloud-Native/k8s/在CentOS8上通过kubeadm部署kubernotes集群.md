# 系统准备
准备三台centos8服务器，主机名与静态IP地址如下表所示（参考下边 master 节点服务器的配置）：

|角色|主机名|ip地址|
|---|---|---|
|master|master|192.168.56.104|
|node|node1|192.168.56.101|
|node|node2|192.168.56.102|

下面以 master 服务器为例，进行相应的配置（node 节点也需要做同样的配置）
1. 查看系统版本
```shell
[root@myk8s-04 ~]# cat /etc/centos-release
CentOS Linux release 8.3.2011
```

2. 关闭防火墙
```shell
[root@myk8s-04 ~]# systemctl stop firewalld
[root@myk8s-04 ~]# systemctl disable firewalld
```

3. 配置网络
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

4. 添加阿里源
```shell
# [root@myk8s-04 ~]# rm -rfv /etc/yum.repos.d/*
[root@myk8s-04 ~]# curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-8.repo
```

5. 配置主机名
```shell
[root@myk8s-04 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

192.168.56.104 myk8s-03.host.com master
192.168.56.102 myk8s-02.host.com node
192.168.56.101 myk8s-01.host.com node
```

6. 关闭selinux
```shell
[root@myk8s-04 ~]# sed -i 's/enforcing/disabled/' /etc/selinux/config
[root@myk8s-04 ~]# setenforce 0
```

7. 关闭swap，注释swap分区
```shell
root@myk8s-04 ~]# swapoff -a
[root@myk8s-04 ~]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Fri Dec 25 03:30:01 2020
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
/dev/mapper/cl-root     /                       xfs     defaults        0 0
UUID=75c36a78-fe92-49b4-a2ff-3a3a7c1aff76 /boot                   ext4    defaults        1 2
# /dev/mapper/cl-swap     swap                    swap    defaults        0 0
```

# 安装常用包和docker-ce
1. 下载docker-ce的repo
```shell
curl https://download.docker.com/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
```

2. 安装依赖（这是相比centos7的关键步骤）
```shell
curl http://docker-release-blue-prod.s3-website-us-east-1.amazonaws.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.4.6-3.1.el8.x86_64.rpm

yum install containerd.io-1.4.6-3.1.el8.x86_64.rpm
```

3. 安装docker-ce
```shell
yum install docker-ce docker-ce-cli

#也可以指定版本安装
yum list docker-ce --showduplicates | sort -r  #查看版本
sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io #安装指定版本
```

4. 启动docker
```shell
systemctl enable docker.service
systemctl start docker.service

#测试验证
docker info
```

5. 安装时遇到问题，CentOS8默认安装了`Podman`，再次安装`docker`时提升冲突，因此需要协助`Podman`和`Buildah`，再次安装`docker`

# 安装kubectl、kubelet、kubeadm
1. 添加阿里kubernetes源
```shell
[root@myk8s-04 ~]# cat /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```

2. 安装
```shell
[root@myk8s-04 ~]# yum -y install kubectl kubelet kubeadm
[root@myk8s-04 ~]# systemctl enable kubelet
```

# 初始化k8s集群
1. kubeadm初始化脚步
```shell
[root@myk8s-04 ~]# kubeadm init --kubernetes-version=1.21.1  \
> --apiserver-advertise-address=192.168.56.104   \
> --image-repository registry.aliyuncs.com/google_containers  \
> --service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
```
POD的网段为: 10.122.0.0/16， api server地址就是master本机IP。

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
记录生成的最后部分内容，此内容是在其它节点加入Kubernetes集群时执行。

2. 根据提示创建`kubectl`
```shell
[root@myk8s-04 ~]# mkdir -p $HOME/.kube
[root@myk8s-04 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@myk8s-04 ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

3. 执行下面命令，使`kubectl`可以自动补充
```shell
[root@myk8s-04 ~]# source <(kubectl completion bash) && echo 'source <(kubectl completion bash)' >> ~/.bashrc
```

4. 查看节点pod
```shell
[root@myk8s-04 ~]# kubectl get node
NAME                STATUS   ROLES                  AGE   VERSION
myk8s-01.host.com   Ready    <none>                 27h   v1.21.1
myk8s-02.host.com   Ready    <none>                 28h   v1.21.1
myk8s-04.host.com   Ready    control-plane,master   28h   v1.21.1
```

# 安装calico网络
1. 安装Calico网络
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

2. 查看pod和node
```shell
[root@myk8s-04 ~]# kubectl get pod --all-namespaces
NAMESPACE              NAME                                         READY   STATUS             RESTARTS   AGE
ingress-nginx          nginx-ingress-controller-6979d75b9d-cbml4    1/1     Running            0          10h
kube-system            calico-kube-controllers-78d6f96c7b-ld74h     1/1     Running            1          12h
kube-system            calico-node-8gjjr                            1/1     Running            1          27h
kube-system            calico-node-j6m97                            1/1     Running            2          28h
kube-system            calico-node-x7jn8                            1/1     Running            21         28h
kube-system            coredns-545d6fc579-mxfcx                     0/1     ImagePullBackOff   0          12h
kube-system            coredns-545d6fc579-tfq84                     0/1     ImagePullBackOff   0          12h
kube-system            etcd-myk8s-04.host.com                       1/1     Running            2          28h
kube-system            kube-apiserver-myk8s-04.host.com             1/1     Running            3          28h
kube-system            kube-controller-manager-myk8s-04.host.com    1/1     Running            2          28h
kube-system            kube-proxy-5ljdp                             1/1     Running            2          28h
kube-system            kube-proxy-nstnq                             1/1     Running            1          28h
kube-system            kube-proxy-nzglk                             1/1     Running            1          27h
kube-system            kube-scheduler-myk8s-04.host.com             1/1     Running            2          28h
kubernetes-dashboard   dashboard-metrics-scraper-856586f554-sztd4   1/1     Running            0          11h
kubernetes-dashboard   kubernetes-dashboard-67484c44f6-58f2z        1/1     Running            0          11h
[root@myk8s-04 ~]# kubectl get node
NAME                STATUS   ROLES                  AGE   VERSION
myk8s-01.host.com   Ready    <none>                 27h   v1.21.1
myk8s-02.host.com   Ready    <none>                 28h   v1.21.1
myk8s-04.host.com   Ready    control-plane,master   28h   v1.21.1
```

# 安装kubernetes-dashboard
1. 安装
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml
# 有问题时可以删除重新创建： kubectl delete -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.3.1/aio/deploy/recommended.yaml

namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```
> 官方部署dashboard的服务没使用nodeport，将yaml文件下载到本地，在service里添加nodeport，修改后的 recommended.yaml 

2. 查看
```shell
[root@myk8s-04 ~]# kubectl get pod -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-856586f554-sztd4   1/1     Running   0          11h
kubernetes-dashboard-67484c44f6-58f2z        1/1     Running   0          11h
```

3. 浏览器访问dashboard：`https://192.168.56.104:30000`
   
4. Token认证
Dashboard 支持 Kubeconfig 和 Token 两种认证方式，我们这里选择Token认证方式登录
有两种方式获取 token
1) 方式一（方便快捷）
```shell
[root@myk8s-04 ~]# kubectl -n kubernetes-dashboard get secret
NAME                               TYPE                                  DATA   AGE
default-token-lvcsn                kubernetes.io/service-account-token   3      11h
kubernetes-dashboard-certs         Opaque                                0      11h
kubernetes-dashboard-csrf          Opaque                                1      11h
kubernetes-dashboard-key-holder    Opaque                                2      11h
kubernetes-dashboard-token-mfz44   kubernetes.io/service-account-token   3      11h
[root@myk8s-04 ~]# kubectl describe secrets -n kubernetes-dashboard kubernetes-dashboard-token-mfz44  | grep token | awk 'NR==3{print $2}'
eyJhbGciOiJSUzI1NiIsImtpZCI6ImhLUDk1UVJCbHl6QWhlQmVvd3ItVEotZ0RxaWNFaVU1dkNfMEZtYm1FdEUifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZC10b2tlbi1tZno0NCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImMyMjc3MTc0LWQ3OGUtNGIwYy1iODczLThkMjgxZmU4NzBhMyIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlcm5ldGVzLWRhc2hib2FyZDprdWJlcm5ldGVzLWRhc2hib2FyZCJ9.Gju8LFlOksjDnHf4DOnyDASbRtlN-N-AzjAZX5lDq23VR_ewc1z2PV2SElY7gTjvPh-usRvITpGGwnNK-8iNKtjP8YSQFBvnjrV1rECVb8tt-Cu_VUN_Nlyokgy6sfoS701-8jV0uLoxwF9eGeI8Skr0mknm_TgKDpC97RtsFeV8zdoY4-_gLBxr6oe1D3XwCZJI-2lDBd8AhL0B9gMxQDs68CN00nZ8SSpcwaOohiSmgQ8eIYC4hJlPYcgOoXqft4mTxAhFLggNNQOIWK5y_1y8WWziu5gsP0lhx2awppMwJ3LX2nqhT6d_FHvrqIzIhzH0Dzfb6cb0SlgJCmmhsA
[root@myk8s-04 ~]#
```
2) 需要生成 admin-user，官方建议   
[官方参考文档](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)
```shell
[root@myk8s-04 ~]# cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF

serviceaccount/admin-user created


[root@myk8s-04 ~]# cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

clusterrolebinding.rbac.authorization.k8s.io/admin-user created


[root@myk8s-04 ~]# kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
Name:         admin-user-token-mwr8p
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 61b3d764-5bf2-431e-99db-d524a4d379a7

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6ImxmV2ZXbWpza3NuUnJlQ0dERjM2Y1prNVZ2VU1JeWZmSTdocWtIR0pMbWMifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLW13cjhwIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiI2MWIzZDc2NC01YmYyLTQzMWUtOTlkYi1kNTI0YTRkMzc5YTciLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.nixHnB5iIUz4x6gzGKwxiUW9ehxVmHP7IR8akrMQ2STvFK0T_v0DCyIVE93tSZxAC6wuUW0NF1QHRYqzIm01xsj9i37NyCCL9ZF-WNw7fjGIKN3FG5ycYRLJ5bzV_rbvmbkm6uC5PYVidR2zw_3w4s2_kiai5Gtwee8BQU0CgbDeZqUKorZR_ZyLLymvjvPMvhMEBxRFgxE-iViAXSbE0xtF2Fq5JYduWlvCEscLAN193RE0GnzLdT7R_6DoiaL5QsU20U_XA1E-3ETUI4HD8zEj89Hok02y_LXB8mEXnU3VjsL2LDeAoIEylhXOfCH8j72kiUCEu7nv3X1NPZ809A
```

# node节点服务器配置以及加入集群
重复 master 节点的 1、2 配置步骤   
第 3 步骤只需安装 kubectl、kubeadm 即可

1. 加入node节点到集群
```shell
kubeadm join 192.168.56.101:6443 --token n9ymj8.nscmc7jz344d9maa \
 --discovery-token-ca-cert-hash sha256:13164b268a07c60fcfe9b71cfba98dc36f630a7c64a471397fd52cf3631f46a7

 kubeadm join 192.168.56.102:6443 --token n9ymj8.nscmc7jz344d9maa \
 --discovery-token-ca-cert-hash sha256:13164b268a07c60fcfe9b71cfba98dc36f630a7c64a471397fd52cf3631f46a7
```

2. 在master上查看节点信息
```shell
[root@myk8s-04 ~]# kubectl get nodes
NAME                STATUS   ROLES                  AGE   VERSION
myk8s-01.host.com   Ready    <none>                 28h   v1.21.1
myk8s-02.host.com   Ready    <none>                 28h   v1.21.1
myk8s-04.host.com   Ready    control-plane,master   29h   v1.21.1
```

3. `node`节点对`kubernetes`进行配置   
拷贝`master`节点服务器的`/etc/kubernetes/admin.conf`到`node`节点服务器的`/etc/kubernetes/`目录下
```shell
[root@myk8s-04 ~]# scp /etc/kubernetes/admin.conf root@192.168.56.101:/etc/kubernetes

[root@myk8s-04 ~]# scp /etc/kubernetes/admin.conf root@192.168.56.102:/etc/kubernetes
```
4. 在节点1执行如下命令
```shell
[root@myk8s-01 ~]# mkdir -p $HOME/.kube
[root@myk8s-01 ~]# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@myk8s-01 ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/confi

[root@myk8s-01 ~]# echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile
[root@myk8s-01 ~]# source ~/.bash_profile
```
> node 节点的 docker 配置文件 /etc/docker/daemon.json 尽量跟 master 节点保持一致   
> 引用：[k8s 集群之使用 kubeadm 在 Centos8 上部署 kubernetes 1.20](https://blog.csdn.net/qq_34596292/article/details/112131042)