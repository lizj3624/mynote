## 网络处理的基本概念

### 硬中断(Hardware Interrupt)

硬件设备产生输入核输出事件的时候，会向CPU发送信号，这个信号就是硬中断。CPU在接收到硬中断之后，会停止现在的工作，以最高优先级处理这个硬件中断事件。

### 软中断(SoftIRQ)

软中断与硬中断在机制上类似只是优先级稍微低一点。在网卡传输数据的过程中，典型的过程是数据到达网卡，网卡通过DMA将数据拷贝到为网卡驱动指定的内存区域，产生硬中断，然后CPU需要自己负责处理中断，将网卡的数据拷贝到内核指定区域，进行TCP/IP协议栈的处理，再通知应用层并将数据拷贝的应用层。在这种情况下，如果一旦一个硬中断来了之后，CPU一直忙碌于处理网卡的这个请求和服务上层应用，那网卡后续数据到来产生的硬中断也会被延后处理，而且其它设备和进程也无法使用这个CPU资源，造成CPU饥饿状态。所以在处理设备硬中断的时候，将整个过程分成两个阶段；

* 第一阶段，CPU只是使用ISR简单而快速响应硬中断，产生一个软中断，然后CPU可以继续进行后续调度处理而网卡可以继续收发数据；

* 第二阶段，CPU会根据scheduler对一系列软中断按照优先级进行排队处理，将数据包移动到TCP/IP协议栈和后续应用程序。

![中断](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-00.png)

### 接收队列

网卡驱动通过DMA将接收到的数据会拷贝到socket buffer并由一个socket descriptor进行标记，所有待处理的数据包的socket descriptor将被存储在接收队列里面，如果有多个接收队列，则网卡会将packet按照某个算法匹配到一个接收队列上。现在的许多网卡都是通过一种叫RSS(Receive Side Scaling)的技术(将数据包的处理任务分配到多个CPU)在硬件上支持Rx多队列的。

在对NIC进行调整的时候，发现RSS(receive side scaling)和RPS(Receive Packet Steering)是两个需要关注的点。RSS和RPS都是网卡为了在接受数据包的时候使用多核架构而进行的性能增强，RSS是在硬件层面而RPS在软件层面。在数据包接收到之后在用户态的处理逻辑怎么处理，应用层的响应数据包如何发送都会影响系统性能，RFS(Receive Flow Steering)和XPS(Transmit Packet Steering)这两个机制就是为了解决这两个问题而产生的。

这四个功能在这个kernel文档里面进行了全面的描述：[这里](https://www.kernel.org/doc/Documentation/networking/scaling.txt)

### RSS(Receive Side Scaling)

当数据包到达NIC之后，将被放到接收队列；在网卡驱动初始化阶段，接收队列会被赋予一个IRQ号，并且会分配一个CPU来处理这个IRQ，这个CPU需要执行这个IRQ的ISR并且在一般情况下还要负责执行后续的数据包在内核阶段的处理。在单核系统中，这是一个很好的工作模型，但是在多核系统中，这种方式在大流量的时候无法发挥多核的作用，只能使用一个内核，进行耗时的TCP/IP协议栈的处理。

所以在现在的NIC中，多数都支持RSS的功能，启用这种功能之后，网卡会有多个接收和发送队列，这些队列对被分配不同的CPU进行处理。

RSS为网卡数据传输使用多核提供了支持，RSS在硬件/驱动级别实现多队列并且使用一个hash函数对数据包进行多队列分离处理，这个hash根据源IP、目的IP、源端口和目的端口进行数据包分布选择，这样同一数据流的数据包会被放置到同一个队列进行处理并且能一定程度上保证数据处理的均衡性。

![rss](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-05.png)

### RFS(Receive Flow Steering)

RPS 全称是 `Receive Packet Steering`, 这是Google工程师 Tom Herbert (therbert@google.com )提交的内核补丁, 在2.6.35进入Linux内核. 这个patch采用软件模拟的方式，实现了多队列网卡所提供的功能，分散了在多CPU系统上数据接收时的负载, 把软中断分到各个CPU处理，而不需要硬件支持，大大提高了网络性能。

RPS是和RSS类似的一个技术，区别在于RSS是网的硬件实现而RPS是内核软件实现。RPS帮助单队列网卡将其产生的SoftIRQ分派到多个CPU内核进行处理。在这个方案中，为网卡单队列分配的CPU只处理所有硬件中断，由于硬件中断的快速高效，即使在同一个CPU进行处理，影响也是有限的，而耗时的软中断处理会被分派到不同CPU进行处理，可以有效的避免处理瓶颈。

![rfs](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-06.png)

### RFS(Receive Flow Steering)

RFS 全称是 `Receive Flow Steering`, 这也是Tom提交的内核补丁，它是用来配合RPS补丁使用的，是RPS补丁的扩展补丁，它把接收的数据包送达应用所在的CPU上，提高cache的命中率。

在使用RPS接收数据包之后，会在指定的CPU进行软中断处理，之后就会在用户态进行处理；如果用户态处理的CPU不在软中断处理的CPU，则会造成CPU cache miss，造成很大的性能影响。RFS能够保证处理软中断和处理应用程序是同一个CPU，这样会保证local cache hit，提升处理效率。

RFS需要和RPS一起配合使用，来达到最好的优化效果，主要是针对单队列网卡多CPU环境(多队列多重中断的网卡也可以使用该补丁的功能，但多队列多重中断网卡有更好的选择:SMP IRQ affinity)

### XPS(Transmit Packet Steering)

XPS通过创建CPU到网卡发送队列的对应关系，来保证处理发送软中断请求的CPU和向外发送数据包的CPU是同一个CPU，用来保证发送数据包时候的局部性。

## 网卡处理收发包过程

接收数据包是一个复杂的过程，涉及很多底层的技术细节，但大致需要以下几个步骤：

1. 网卡收到数据包。
2. 将数据包从网卡硬件缓存转移到服务器内存中。
3. 通知内核处理。
4. 经过TCP/IP协议逐层处理。
5. 应用程序通过`read()`从`socket buffer`读取数据。

![01](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-01.png)

### 将网卡收到的数据包转移到主机内存（NIC与驱动交互）

NIC在接收到数据包之后，首先需要将数据同步到内核中，这中间的桥梁是`rx ring buffer`。它是由NIC和驱动程序共享的一片区域，事实上，`rx ring buffer`存储的并不是实际的packet数据，而是一个描述符，这个描述符指向了它真正的存储地址，具体流程如下：

1. 驱动在内存中分配一片缓冲区用来接收数据包，叫做`sk_buffer`；
2. 将上述缓冲区的地址和大小（即接收描述符），加入到`rx ring buffer`。描述符中的缓冲区地址是DMA使用的物理地址；
3. 驱动通知网卡有一个新的描述符；
4. 网卡从`rx ring buffer`中取出描述符，从而获知缓冲区的地址和大小；
5. 网卡收到新的数据包；
6. 网卡将新数据包通过DMA直接写到`sk_buffer`中。

![02](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-02.png)

当驱动处理速度跟不上网卡收包速度时，驱动来不及分配缓冲区，NIC接收到的数据包无法及时写到`sk_buffer`，就会产生堆积，当NIC内部缓冲区写满后，就会丢弃部分数据，引起丢包。这部分丢包为`rx_fifo_errors`，在`/proc/net/dev`中体现为fifo字段增长，在ifconfig中体现为overruns指标增长。

### 通知系统内核处理（驱动与Linux内核交互）

这个时候，数据包已经被转移到了`sk_buffer`中。前文提到，这是驱动程序在内存中分配的一片缓冲区，并且是通过DMA写入的，这种方式不依赖CPU直接将数据写到了内存中，意味着对内核来说，其实并不知道已经有新数据到了内存中。那么如何让内核知道有新数据进来了呢？答案就是中断，通过中断告诉内核有新数据进来了，并需要进行后续处理。

提到中断，就涉及到硬中断和软中断，首先需要简单了解一下它们的区别：

- 硬中断： 由硬件自己生成，具有随机性，硬中断被CPU接收后，触发执行中断处理程序。中断处理程序只会处理关键性的、短时间内可以处理完的工作，剩余耗时较长工作，会放到中断之后，由软中断来完成。硬中断也被称为上半部分。
- 软中断： 由硬中断对应的中断处理程序生成，往往是预先在代码里实现好的，不具有随机性。（除此之外，也有应用程序触发的软中断，与本文讨论的网卡收包无关。）也被称为下半部分。

**当NIC把数据包通过DMA复制到内核缓冲区`sk_buffer`后，NIC立即发起一个硬件中断。CPU接收后，首先进入上半部分，网卡中断对应的中断处理程序是网卡驱动程序的一部分，之后由它发起软中断，进入下半部分，开始消费`sk_buffer`中的数据，交给内核协议栈处理。**

![03](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-03.png)

通过中断，能够快速及时地响应网卡数据请求，但如果数据量大，那么会产生大量中断请求，CPU大部分时间都忙于处理中断，效率很低。为了解决这个问题，现在的内核及驱动都采用一种叫NAPI（new API）的方式进行数据处理，其原理可以简单理解为 中断+轮询，在数据量大时，一次中断后通过轮询接收一定数量包再返回，避免产生多次中断。

整个中断过程的源码部分比较复杂，并且不同驱动的厂商及版本也会存在一定的区别

![04](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-04.png)

### NUMA-SMP架构

随着单核CPU的频率在制造工艺上的瓶颈，CPU制造商的发展方向也由纵向变为横向：从CPU频率转为每瓦性能。CPU也就从单核频率时代过渡到多核性能协调。

SMP(对称多处理结构)：即CPU共享所有资源，例如总线、内存、IO等。

SMP 结构：一个物理CPU可以有多个物理Core，每个Core又可以有多个硬件线程。即：每个HT有一个独立的L1 cache，同一个Core下的HT共享L2 cache，同一个物理CPU下的多个core共享L3 cache。

下图(摘自[内核月谈](https://mp.weixin.qq.com/s/y1NSE5xdh8Nt5hlmK0E8Og))中，一个x86 CPU有4个物理Core，每个Core有两个HT(Hyper Thread)。

![07](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-07.png)

#### NUMA 架构

在前面的FSB(前端系统总线)结构中，当CPU不断增长的情况下，共享的系统总线就会因为资源竞争(多核争抢总线资源以访问北桥上的内存)而出现扩展和性能问题。

在这样的背景下，基于SMP架构上的优化，设计出了NUMA(Non-Uniform Memory Access)非均匀内存访问。

内存控制器芯片被集成到处理器内部，多个处理器通过QPI链路相连，DRAM也就有了远近之分。(如下图所示：摘自[CPU Cache](http://mechanical-sympathy.blogspot.com/2013/02/cpu-cache-flushing-fallacy.html))

CPU 多层Cache的性能差异是很巨大的，比如：L1的访问时长1ns，L2的时长3ns…跨node的访问会有几十甚至上百倍的性能损耗。

![smp-08](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/smp-08.png)

#### NUMA 架构下的中断优化

这时我们再回归到中断的问题上，当两个NUMA节点处理中断时，CPU实例化的`softnet_data`以及驱动分配的`sk_buffer`都可能是跨Node的，数据接收后对上层应用Redis来说，跨Node访问的几率也大大提高，并且无法充分利用L2、L3 cache，增加了延时。

同时，由于`Linux wake affinity`特性，如果两个进程频繁互动，调度系统会觉得它们很有可能共享同样的数据，把它们放到同一CPU核心或`NUMA Node`有助于提高缓存和内存的访问性能，所以当一个进程唤醒另一个的时候，被唤醒的进程可能会被放到相同的`CPU core`或者相同的NUMA节点上。此特性对中断唤醒进程时也起作用，在上一节所述的现象中，所有的网络中断都分配给`CPU 0`去处理，当中断处理完成时，由于`wakeup affinity`特性的作用，所唤醒的用户进程也被安排给`CPU 0`或其所在的numa节点上其他core。而当两个`NUMA node`处理中断时，这种调度特性有可能导致Redis进程在`CPU core`之间频繁迁移，造成性能损失。

综合上述，将中断都分配在同一`NUMA Node`中，中断处理函数和应用程序充分利用同NUMA下的L2、L3缓存、以及同Node下的内存，结合调度系统的`wake affinity`特性，能够更进一步降低延迟。

## 设置软中断绑定多核的步骤

软中断绑定多核的脚步：

```shell
# cat set_irq_affinity.sh
 
# setting up irq affinity according to /proc/interrupts
# 2008-11-25 Robert Olsson
# 2009-02-19 updated by Jesse Brandeburg
#
# > Dave Miller:
# (To get consistent naming in /proc/interrups)
# I would suggest that people use something like:
#       char buf[IFNAMSIZ+6];
#
#       sprintf(buf, "%s-%s-%d",
#               netdev->name,
#               (RX_INTERRUPT ? "rx" : "tx"),
#               queue->index);
#
#  Assuming a device with two RX and TX queues.
#  This script will assign:
#
#       eth0-rx-0  CPU0
#       eth0-rx-1  CPU1
#       eth0-tx-0  CPU0
#       eth0-tx-1  CPU1
#
 
set_affinity()
{
    if [ $VEC -ge 32 ]
    then
         MASK_FILL=""
         MASK_ZERO="00000000"
         let "IDX = $VEC / 32"
         for ((i=1; i<=$IDX;i++))
         do
             MASK_FILL="${MASK_FILL},${MASK_ZERO}"
         done
 
         let "VEC -= 32 * $IDX"
         MASK_TMP=$((1<<$VEC))
         MASK=`printf "%X%s" $MASK_TMP $MASK_FILL`
     else
         MASK_TMP=$((1<<(`expr $VEC + $CORE`)))
         MASK=`printf "%X" $MASK_TMP`
     fi
 
     printf "%s mask=%s for /proc/irq/%d/smp_affinity\n" $DEV $MASK $IRQ
     printf "%s" $MASK > /proc/irq/$IRQ/smp_affinity
}
 
if [ $# -ne 2 ] ; then
    echo "Description:"
    echo "    This script attempts to bind each queue of a multi-queue NIC"
    echo "    to the same numbered core, ie tx0|rx0 --> cpu0, tx1|rx1 --> cpu1"
    echo "usage:"
    echo "    $0 core eth0 [eth1 eth2 eth3]"
    exit
fi
 
CORE=$1
 
# check for irqbalance running
IRQBALANCE_ON=`ps ax | grep -v grep | grep -q irqbalance; echo $?`
if [ "$IRQBALANCE_ON" == "0" ] ; then
    echo " WARNING: irqbalance is running and will"
    echo "          likely override this script's affinitization."
    echo "          Please stop the irqbalance service and/or execute"
    echo "          'killall irqbalance'"
fi
 
#
# Set up the desired devices.
#
shift 1
 
for DEV in $*
do
    for DIR in rx tx TxRx
    do
        MAX=`grep $DEV-$DIR /proc/interrupts | wc -l`
        if [ "$MAX" == "0" ] ; then
            MAX=`egrep -i "$DEV:.*$DIR" /proc/interrupts | wc -l`
        fi
        if [ "$MAX" == "0" ] ; then
            echo no $DIR vectors found on $DEV
            continue
        fi
        
        for VEC in `seq 0 1 $MAX`
        do
            IRQ=`cat /proc/interrupts | grep -i $DEV-$DIR-$VEC"$"  | cut  -d:  -f1 | sed "s/ //g"`
            if [ -n  "$IRQ" ]; then
                set_affinity
            else
                IRQ=`cat /proc/interrupts | egrep -i $DEV:v$VEC-$DIR"$"  | cut  -d:  -f1 | sed "s/ //g"`
                if [ -n  "$IRQ" ]; then
                set_affinity
                fi
            fi
        done
    done
done
```

脚本参数是：set_irq_affinity.sh core eth0 [eth1 eth2 eth3]
可以一次设置多个网卡，core的意思是从这个号开始递增。

由于有硬件中断平衡，所以我们也不需要rps了，用以下脚本关闭掉：

```shell
# cat em.sh
#! /bin/bash                                                                             
for i in `seq 0 7`
do
    echo 0|tee  /sys/class/net/em1/queues/rx-$i/rps_cpus >/dev/null
    echo 0|tee  /sys/class/net/em2/queues/rx-$i/rps_cpus >/dev/null
done
#./em.sh
```

最后一定要记得关掉无用的关闭irqbalance, 除了捣乱没啥用途:

```shell
service irqbalance stop
```

## 查看网卡多队列的常用的命令

### 网卡是否支持多队列

通过lspci方式查看网卡信息，如果有MSI-X， Enable+ 并且Count > 1，则该网卡是多队列网卡，多队列网卡内部会有多个 Ring Buffer。

```shell
[root@localhost ~]# lspci -vvv | grep -A50 "Ethernet controller" | grep -E "Capabilities|Ethernet controller"
01:00.0 Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)
	Capabilities: [40] Power Management version 3
	Capabilities: [50] MSI: Enable- Count=1/1 Maskable+ 64bit+
	Capabilities: [70] MSI-X: Enable+ Count=10 Masked-
	Capabilities: [a0] Express (v2) Endpoint, MSI 00
	Capabilities: [e0] Vital Product Data
```

### 网卡支持最大队列数及当前使用队列

```shell
[root@localhost ~]# ethtool -l eth0
Channel parameters for eth0:
Pre-set maximums:
RX:		0
TX:		0
Other:		1
Combined:	63
Current hardware settings:
RX:		0
TX:		0
Other:		1
Combined:	40
```

### CPU的个数

```shell
[root@localhost ~]# cat /proc/cpuinfo | grep processor | wc -l
40
```

### 查看网卡信息
```shell
#常用1
ifconfig

#常用2
lspci |grep -i 'eth'
lspci | grep -i net

#常用3
ethtool eth0
ethtool -p eth0

```

### 修改网卡队列数

```shell
ethtool -L eth0 combined 8

ethtool -L eth0 rx 8
ethtool -L eth0 tx 8
```

设置后可以在/sys/class/net/eth0/queues/目录下看到对应的队列

```shell
[root@localhost ~]# cd /sys/class/net/eth0/queues/
[root@localhost queues]# ls
rx-0  rx-2  rx-4  rx-6  tx-0  tx-2  tx-4  tx-6
rx-1  rx-3  rx-5  rx-7  tx-1  tx-3  tx-5  tx-7
```

### 查看CPU软中断

```shell
mpstat -P ALL 2 10

top

##查看软中断
cat /proc/softirqs|tr -s ' ' '\t'|cut -f 1-8

##硬件中断
cat /proc/interrupts

##中断号绑定CPU核
cat /proc/irq/$(中断号)/smp_affinity
```



## 引用

* [Redis 高负载下的中断优化](https://tech.meituan.com/2018/03/16/redis-high-concurrency-optimization.html)
* [MYSQL数据库网卡软中断不平衡问题及解决方案-褚霸](http://blog.yufeng.info/archives/2037)

* [网卡多队列总结](https://xusenqi.github.io/2018/11/23/%E7%BD%91%E5%8D%A1%E5%A4%9A%E9%98%9F%E5%88%97%E6%80%BB%E7%BB%93/)

* [容器云负载均衡之三：RSS、RPS、RFS和XPS调整](https://blog.csdn.net/cloudvtech/article/details/80182074)

