对于网络地址转换(NAT)、桥接模式、Host-Only模式不熟悉的同学，可以参考文章：[VMware Workstation环境下的三种网络配置桥接，NAT和HOST-ONLY](https://blog.csdn.net/xybelieve1990/article/details/53063774)

对于virtualbox虚拟机，我们最常用的网络方式可能就要数网络地址转换（NAT）了，基本上不需要什么额外配置虚拟机就可以访问外网了，设置端口转发也可以很容易实现真机访问虚拟机，但想实现虚拟机和真机，以及虚拟机之间的通信就比较难了，看到网上的解决方案是虚拟机使用两块网卡，一块使用 NAT模式，实现虚拟机访问外网，一块使用Host-Only模式，实现虚拟机与虚拟机之间以及虚拟机与真机之间的通信。因为Host-Only会在真机上虚拟出一块网卡，并且会给虚拟机分配独立的内网ip，相当于为所有虚拟机和真机组建了一个局域网，并且可以设置固定的ip地址，而桥接模式虽然也能分配独立ip，但通常都是动态分配的，使用很不方便。我们也可以在需要连接外网和需要主机与虚拟机连接两种情况下切换网络设置重启网卡，但是这样很麻烦，所以需要添加一个虚拟网卡配合主机自身的网卡通过设置完成我们的需要。

注意：虚拟机的所有设置必须在关机情况下完成，特别是新增的虚拟网卡，如果不在关机状态下，则无法进行网卡2的操作；如果只是改了虚拟机的网络设置，则可以通过重启网卡进行生效。

centos6的网卡重启方法：`service network restart`
centos7的网卡重启方法：`systemctl restart network`

## 一、通过网络地址转换(NAT)配合host-only
### 设置步骤

#### 创建虚拟网卡

(以mac为例，windows系统有些相关设置选项)

在virtualbox的管理-->偏好设置-->网络-->仅主机（Host-Only）网络添加一块网卡，点击右边的小加号即可，该网卡为虚拟网卡，点击小笔头可以看到ip地址。

![01](https://github.com/lizj3624/mynote/blob/master/Cloud-Native/mac-vbox/picture/mac-vx-04.jpg)

![01](https://github.com/lizj3624/mynote/blob/master/Cloud-Native/mac-vbox/picture/mac-vx-01.jpg)

这时候如果你在MacBook的终端中使用ifconfig命令查看，你会发现，多出来一个vboxnet0的网卡，ip地址就是192.168.56.1

#### 设置virtualBox虚拟机中的网卡

(首先设置真机网卡-网卡1)

在virtualBox选择相应虚拟机系统->设置->网络->网卡1 ->网络地址转换(NAT)，这里表示虚拟机系统公用真机系统网络，设置完成后可以在虚拟机中上网，虚拟机可以ping通真机ip，但是真机无法ping通虚拟机，多个虚拟机之间也无法ping通。

![02](https://github.com/lizj3624/mynote/blob/master/Cloud-Native/mac-vbox/picture/mac-vx-02.jpg)

#### 设置virtualBox虚拟机中的网卡

(设置手动添加的虚拟网卡-网卡2)
在virtualBox选择相应虚拟机系统->设置->网络->网卡2 -> 仅主机(Host-Only)适配器，界面名称选择手动添加的虚拟网卡，完成后主机和虚拟机、虚拟机和虚拟机即可以用该虚拟网卡通信。

![03](https://github.com/lizj3624/mynote/blob/master/Cloud-Native/mac-vbox/picture/mac-vx-03.jpg)

 

### 通过桥接模式实现虚拟机上网，主机和虚拟机、虚拟机之间互通
只需要修改虚拟机的ip和主机在同一个网段即可。

修改命令：

1）ifconfig 网卡名 ip地址，命令模式修改在重启后会失效

2）如果是centos图形化可以在设置->网络设置中找到ipv4选择manual手动设置ip地址模式

3）也可以直接在/etc/sysconfig/network-scrips/ 目录下找到对应网卡的配置文件进行配置。

### 虚拟机ssh端口及防火墙配置
VirtualBox默认的分辨率非常低，可以通过安装增强工具进行优化。不过由于我们不需要图形化界面，其实可以通过其他方式解决这一问题，就是用xshell或者putty通过ssh远程登陆到虚拟机上，mac电脑可以直接通过ssh命令。

* 打开ssh服务

```shell
service sshd start
chkconfig sshd on
```


分别启动ssh服务，并将ssh设定为自启动

* 关闭防火墙

```shell
# centos7的防火墙操作和之前版本区别很大：

sudo systemctl stop firewalld.service
sudo systemctl disable firewalld.service
```



### 引用

1. [在Mac上利用VirtualBox搭建本地虚拟机环境的方法](https://m.yisu.com/zixun/150751.html)

2. [virtual实现上网及主机和virtual虚拟机之间的互通](https://blog.csdn.net/xybelieve1990/article/details/86736961)