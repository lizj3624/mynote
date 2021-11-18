路由协议，套用IT里面的术语，实际上就是分布式数据库系统，它包含了节点间的数据传递和节点内的数据处理。对于BGP来说，节点间基于TCP（端口179）的连接，在这个基础上，可以构建AS间的EBGP，AS内的IBGP，IBGP有full mesh，BGP路由反射器等，这些都是BGP节点之间的连接方式，这次看看BGP router内部是如何处理数据。

BGP是一种path vector路由协议，对比其他类的路由协议，path vector随路由携带的辅助信息更多，处理也稍微复杂一些。BGP内部处理流程简单的画了一下，如下所示，各家的实际实现可能略有不同，但应该大同小异。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-01.jpg)

在看这个处理流程前，先看看一些相关概念。

## **Path Attribute（PA）**

Path对应的就是route，那顾名思义，这是BGP Route的一些参数属性。**Path Attribute是BGP的基础组成元素，它贯穿了整个BGP 路由处理的过程。**

首先，BGP节点间传递的BGP message就是由NLRI（Network Layer Reachability Information）和PA组成。这个可以从BGP Update Message看出来。如果只考虑IP路由，那么NLRI就是IP prefix。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-02.jpg)

其次，BGP policy engine的处理是围绕着Path Attribute展开的。第三，BGP best path selection，是根据PA做的算法。

Path attribute分为4类：

1. Well-known Mandatory: 所有的BGP router必须识别这个属性，并且所有的BGP Message必须包含这个属性
2. Well-known Discretionary: 所有的BGP router必须识别这个属性，BGP Message可以不包含这个属性
3. Optional Transitive: BGP router可以不识别这个属性，如果不识别直接无视这个属性
4. Optional Non-transitive: BGP router可以不识别这个属性，如果不识别要将这条BGP Message丢弃

常见的BGP Path Attribute如下表所示：

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-03.jpg)

这里NEXT_HOP也属于Path Attribute，BGP处理过程中可以修改NEXT_HOP。EBGP router默认会修改NEXT_HOP为自己，IBGP router默认不会修改NEXT_HOP，这在上一篇讲过。

## **Routing Information Base（RIB）**

RIB其实是设备商的术语。或许不太恰当，但是Global RIB可以对应操作系统里面的路由表。Global RIB和路由表都决定IP packet的三层转发的路径。RIB除了存放路由条目，还保存一些路由协议相关的辅助信息。除了Global RIB，每个路由协议都有自己的RIB，这样，路由协议可以将一些生（Raw）数据与真正应用的数据进行隔离。BGP维护几个RIB，包括了：

- BGP Adjacent In RIB：保存所有接收到的BGP Message，这里可能存在多条BGP Message指向同一个目的IP prefix
- BGP Local RIB：保存经过处理和运算得到的最优BGP Message，对于同一个目的IP prefix，只存在一条最优的BGP Message
- BGP Adjacent out RIB：保存将要发送给BGP Peer的BGP Message

BGP协议收发的数据不会直接写到Global RIB里，而是放到了BGP自己的RIB里面，在适当的时候写入Global RIB，前面说过，这样可以实现数据隔离，有选择的将BGP数据写入主路由表。BGP的三个RIB保存着不同处理阶段的BGP Message，为不同阶段的操作提供数据。接下来过一下BGP路由处理过程。

## **Route Processing**

## *1. BGP Adjacent in RIB*

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-04.jpg)

这一步比较简单，来者不拒，所有收到的BGP Message都存到了BGP Adjacent in RIB。

## *2. Input Policy*

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-05.jpg)

Input policy会完成两部分工作，filtering和manipulation。

Filtering会根据Path Attribute过滤BGP Message，这里需要注意两个内置的过滤，一个是判断当前的AS是否在BGP Message的AS_PATH中，如果在的话，那么这是一条之前已经经过当前AS的Message，这条Message会被过滤。另一个会判断BGP Message里的NEXT_HOP是否可达，如果不可达，那么这条Message会被标成Invalid，也会被过滤。除了内置的过滤，用户（对，就是网工）和控制程序也可以添加过滤规则，例如通过route-map，access-list，distribution-list等。

Manipulation会修改BGP Message的Path Attribute，这样可以控制后面的步骤，例如Best path selection。举个例子，BGP Router从两个邻居收到同一个IP Prefix的BGP Message，那么可以通过修改某一个邻居的BGP Message的PA，使得其中一条BGP Message在下一步中胜出。

## *3. Best path Selection*

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-06.jpg)

Local Route是本地的并且希望通过从BGP发布出去的路由。例如思科的设备，通过network命令可以发布本地路由，也可以通过redistribution，将IGP的路由重分布到BGP。这些Local Route都将转换成了BGP Message，和经过Input policy过滤和修改过的BGP Message，一起参与Best Path Selection。

Best Path Selection是一个根据Path Attribute运算，从指向同一个目的IP prefix的，多条BGP Message中选出一条最优的过程。这个过程不复杂，但是比较繁琐，相应的介绍也很多了，限于篇幅我就不展开了，感兴趣可以看看思科的文档[BGP Best Path Selection Algorithm](https://link.zhihu.com/?target=https%3A//www.cisco.com/c/en/us/support/docs/ip/border-gateway-protocol-bgp/13753-25.html)。

## *4. BGP Local RIB*

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-07.jpg)

Best Path Selection能确保指向同一个IP prefix只有一条（其实也可以多条，取决于multipath）最优的BGP Message，这些BGP Message会存到BGP Local RIB。接下来的处理会分两条路径。

第一个是写入到Global RIB，也就是全局路由表。当路由器中，没有其他的路由协议生成了指向相同目的IP prefix的路由，或者有的话，该路由协议的Administrative Distance（AD）大于BGP的AD值，那么这个时候BGP Local RIB的路由才会写入到Global RIB。EBGP的AD值是20，小于大部分路由协议，IBGP的AD值是200，大于大部分路由协议。

第二个是输出到Output Policy，进而发往其他的BGP Peer。

这两个路径互不影响，就算BGP没有竞争过其他路由协议，没有将路由写到全局路由表，也不影响路由传递给其他的BGP Peer。

## *5. Output Policy*

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-08.jpg)

与Input Policy类似，这里也做filtering和manipulation。

Filtering会根据Path Attribute过滤BGP Message，可以自己定义，也有BGP程序自带的过滤。还是以AS_PATH为例，如果目的BGP Peer的AS在BGP Message的AS_PATH中，那么这条BGP Message不会生成对应的发往该BGP Peer的BGP Message。

Manipulation会修改BGP Message的Path Attribute，例如修改MED值，进而生成发往BGP Peer的BGP Message。

经过Output Policy之后，一条BGP Message，会生成针对每一个可以送达的BGP Peer的，多条BGP Message。虽然来自同一个BGP Message，但是这里的每个BGP Message里面包含的Path Attribute可能因为定义策略不一样。

## *6. BGP Adjacent out RIB*

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/bgp-router-process-09.jpg)

类似于第一步，这部分也简单，生成好的BGP Message发往对端的BGP Peer。

## **最后**

以上就是BGP路由处理过程，可以看出都是围绕Path Attribute。如果说BGP基于TCP传输，给BGP router间的传输带来可靠性，那么Path Attribute给BGP的应用带来了灵活性。