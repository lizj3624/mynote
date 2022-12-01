---
title: "DPDK在四层LB中的应用"
date: 2022-12-01T12:31:13+08:00
tags:
   - dpdk 
   - 四层LB 
categories:
   - dpdk 
   - 四层LB 
toc: true
---

目前我们使用四层高性能的LB是基于DPDK的LVS，下面称为DPDK-LVS，[DPDK](https://www.dpdk.org/)是Intel开源的用户态网高性能的络数据包开发包，[官方测试](http://fast.dpdk.org/doc/perf/DPDK_18_08_Intel_NIC_performance_report.pdf)512K的数据包，PPS可以达到千万级别。下面我们剖析一下DPDK高性能的原因。

## 一、bypass内核

### 1、UIO技术(用户态IO)

**DPDK网络收发包可以运行在用户态，主要依赖的UIO技术，UIO（Userspace I/O）是Linux运行在用户空间的I/O技术，可以从用户态直接读取网卡寄存器中的网络数据包，从而绕过处理复杂的Linux网络协议栈，提升网络收发包。**

![01](./uio.png)

左边是内核协议栈：

**网卡 -> 驱动 -> 协议栈 -> Socket接口 -> 业务**

右边是DPDK的方式（基于UIO（Userspace I/O）旁路数据）：

**网卡 -> DPDK轮询模式-> DPDK基础库 -> 业务(lvs)**

由于使用UIO技术，因此在安装DPDK-LVS时，需要先装载UIO的内核模块(**igb_uio.ko**)，DPDK-LVS安装时还必须装载另一个kni内核模块(**rte_kni.ko**)，这个模块主要是dpdk通过这个内核模块与内核交互。

### 2、PMD模式(主动轮询)

DPDK通过主动轮询(PMD，Poll Mode Driver)机制去掉网卡的硬中断，PMD可以极大提升了网卡 I/O 性能。但是这与内核基于中断模式收发包不兼容，因此DPDK实现了基于轮询方式的 PMD网卡驱动。

该驱动由用户态的 API 以及 PMD Driver 构成，内核态的 UIO Driver 屏蔽了网卡发出的中断信号，然后由用户态的 PMD Driver 采用主动轮询的方式。

运行在PMD的Core会处于用户态CPU 100%的状态，因此我们top看到DPDK-LVS的CPU使用率都是100%。

> DPDK-LVS开启多核多线程模式，top看到的cpu使用率是核数*100%。
> 
> 网络空闲时CPU长期空转，会带来能耗问题。所以，DPDK推出Interrupt DPDK模式，以后可以改成这个模式，节约资源。
> 
> 由于dpdk实现了PMD的网卡驱动，更换新网卡时需要查看这个DPDK-LVS引用的DPDK是否支持这个网卡，适配25G mlx网卡时将DPDK升级18.08版本，因为从这个版本才开始支持25G mlx网卡驱动。

**DPDK通过UIO技术旁路了内核，通过PMD机制避免了硬中断， 从而可以在用户态进行收发包的处理。**

## 二、HugePage(大页内存)

现在Linux内核主要是通过**虚拟内存**进行内存管理，将虚拟内存和物理内存分成大小相同的很多份，每份称为**页**，一般一页大小4k，也有2M，4M，1G的大页，通过页表查找进行虚拟内存和物理内存之间映射，

内存很大时通过多级页表查找进行寻址。由于通过多级页表查找寻址，可能影响寻址性能，因此加入了**TLB**(Translation Lookaside Buffer)cache机制，TLB是缓存页表，避免多级页表查找，DPDK引入大页主要是

避免TLB miss。

![02](./v-m.jpg)
![03](./v-m.jpg)

> 因为使用了大页，DPDK-LVS在启动前必须进行大页设置
> 
> echo 15000 > /sys/devices/system/node/$NODENAME_A/hugepages/hugepages-2048kB/nr_hugepages
> 
> echo 3000 > /sys/devices/system/node/$NODENAME_B/hugepages/hugepages-2048kB/nr_hugepages

## 三、多核技术

![04](./m-cpu.png)
![05](./m-cpu01.jpeg)

### 1、RSS(网卡多队列)

随着CPU进入多核时代，为适配CPU的多核，从而提升网卡吞吐量，网卡支持多队列，也就是RSS(Receive Side Scaling)技术，RSS是一种能够在多处理器系统下使接收报文在多个CPU之间高效分发的网卡驱动技术。

### 2、NUMA亲和

![06](./numa.png)

随着CPU进入多核时代，多核CPU通过一条数据总线访问内存延迟很大，因此NUMA 架构应运而生，NUMA架构全称为非一致性内存架构 (Non Uniform Memory Architecture)，与 SMP 中的 UMA 统一寻址内存架构相对。

在 NUMA 系统中有本地内存与远端内存的区别。访问本地内存有更小的延迟和更大的带宽，跨处理器内存访问速度会相对较慢一点，但是整个内存对于所有的处理器都是可见的。  

```shell
## 通过lscpu命令查询NUMA的CPU核数
$ lscpu
...
NUMA node0 CPU(s): 0-7       # numa0上是0-7核
NUMA node1 CPU(s): 8-15      # numa1上是8-15核
...
```

> DPDK-LVS只用了NUMA 0，也只用NUMA 0上的CPU核，避免跨NUMA访问

### 3、无锁

为适配多核多线程，DPDK开发高性能的无锁环性队列(rte_ring)，提升数据包的处理性能。

![07](./ring.png)

## 四、DPDK-LVS的包处理流程

### 1、DPDK-LVS的底层收发包

DPDK实现一套网络数据包处理的API接口，以代替Linux内核协议栈的socket接口，DPDK-LVS底层调用DPDK网络数据包处理的API，从而实现高性能网络数据包处理。

![08](./dpdk-arch.png)

DPDK核心模块：

a、环境抽象层 (EAL)，环境抽象层提供了一个通用接口，该接口对应用程序和库隐藏了环境细节，提供了收发包和中断轮询等核心API。

b、内存池管理器 (librte_mempool)，提供共享内存管理的API，网络数据包数据接口mbuf(类似内核skb数据结构)就是基于这个API实现的。

c、环管理器 (librte_ring)，提供无锁的环形队列API。

### 2、DPDK-LVS的分流

DPDK-LVS启动多个工作线程(worker)，工作线程的个数跟配置核数一致，并通过CPU亲和性绑定到工作线程上。

启动多个工作线程的DPDK-LVS服务的入流量通过**RSS**进行分流，针对FullNat模式下返回包通过**Flow Director**技术返回到相同的工作线程中。

a、入流量(inbonding)通过RSS分流，主要依赖入包的五元组hash后负载到工作线程中。

![09](./rss-dpdk.png)

b、FullNat模式时，不同的工作线程绑定到不同的local IP，返回包进入网卡时，根据返回包的local IP，通过**Flow Director机制**将返回包定向负载到入包时的工作线程。

![10](./fdir.jpeg)

### 3、DPDK-LVS的调度

DPDK-LVS将入包负载到工作线程后，根据入包的五元组查找Session缓存，命中后复用原有的Session，没有命中时通过负载算法挑选后端实例，创建新Session。

这里的调度算法主要复用原生LVS的调度算法(wrr，ip_hash)等。

### 4、DPDK-LVS的路由

DPDK-LVS挑选后端实例，创建Session后，将网络包转发到后端实例，转发时依赖网络路由(route)。DPDK-LVS启动时加入相应的路由信息。下图是DPDK-LVS启动后加入一个默认路由和直连路由。

> DR模式时，需要依赖直连路由，直连路由是根据配置中网卡信息计算出来网络路由,
> 默认路由是根据配置设置的。

## 五、引用

1、[深入理解DPDK程序设计|Linux网络2.0](https://mp.weixin.qq.com/s?__biz=MzkyMTIzMTkzNA==&mid=2247523649&idx=1&sn=5bcdd0efff2d2322df4af877ea61bfcd&chksm=c1846a10f6f3e3061cb336a623a28ec04cedb002f5bf5293175412b0973119d4625fac868870&scene=21#wechat_redirect)

2、[DPDK大页内存原理](https://xie.infoq.cn/article/a7c83189a19387018b8595e98)

3、[深入浅出DPDK-笔记](https://rtoax.blog.csdn.net/article/details/109370979)

4、[DPDK之KNI原理](https://cloud.tencent.com/developer/article/1418603)

5、[DPDK KNI原理和实现](http://blog.chinaunix.net/uid-28541347-id-5856227.html)

6、[DPDK全面分析](https://blog.csdn.net/majianting/article/details/123424912)

7、[DPDK多线程初步解析](https://www.jianshu.com/p/90f0f5e42dcd)

8、[DPDK核心库详解](https://dpdk-docs.readthedocs.io/en/latest/prog_guide/index.html)
