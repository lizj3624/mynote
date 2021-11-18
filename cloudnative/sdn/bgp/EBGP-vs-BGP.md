在上一篇（[BGP漫谈](https://zhuanlan.zhihu.com/p/25433049)）介绍了BGP的一些基本概念。我们知道了BGP分为EBGP和IBGP。这次再进一步看看EBGP和IBGP有什么区别。

## **应用场景**

从应用场景看，EBGP和IBGP的区别还是很明显的。

EBGP连接了互联网上一个个相对独立的AS（Autonomous system）。什么是AS？AS就是一个独立管理的网络。大公司或者组织的网络一般是由一到多个AS组成，小的公司或者个人是通常会接入到ISP（Internet Service Provider）的AS。不管怎么样，如果设备在互联网上，那么必定是属于某一个AS。EBGP是用来连接各个AS，这样互联网上的设备的才能够彼此互连。AS之间的连接协议，目前在用的，有且仅有EBGP一种。

IBGP应用在AS内部，作为IGP的一种。一般的IGP，例如OSPF，EIGRP，用来在邻接路由器之间传递路由。而IBGP可以用来在edge router之间同步路由，edge router并不需要邻接。Edge router是指在AS边缘，用来连接其他AS的router，那么edge router肯定是运行了EBGP。同时这个edge router也会有对端AS的路由。通过IBGP，edge router会将学习到的对端AS的路由，传递给其他的edge router。这样，可以实现跨AS的连通。例如下图中，AS11111和AS33333都连接到了AS22222。AS22222的两个Edge router交换彼此的路由，这样AS11111要访问AS33333，先将包发给AS22222左边的edge router。由于交换了路由，AS22222左边的edge router会把包发给右边的edge router，右边的edge router再发给AS33333。注意，AS22222里的两个edge router不用直连，只需要彼此路由可达即可。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/ebgp-vs-ibgp-01.jpg)

这里运行IBGP的好处是：

- 路由不需要配置重分布，只需要建立好EBGP和IBGP连接，那么相应的路由会通过BGP协议传递。
- 对于路由器来说，EBGP和IBGP只是对应的参数不太一样，但都是通过一个BGP进程来运行，这不会增加路由器的负担
- 中介路由器（intermediate router）不需要关心AS以外的路由信息（例如AS11111和AS22222的路由都不会出现在中介路由器上），这样中介路由器的路由表可以比较小。

虽然应用在AS内部，这里的IBGP仍然起的是连接互联网的作用。

当然，IBGP也可以当成一个普通的IGP，用来在邻接路由器之间传递路由。由于基于TCP协议，当网络规模足够大的时候，IBGP相比OSPF能更加稳定可靠的传输路由信息。

## **技术细节**

从技术细节看，EBGP和IBGP的区别不是那么明显。

EBGP和IBGP都遵循BGP协议，它们的工作流程，处理方式，甚至核心程序，都是一样的。区别在于一些细节参数，默认行为不同。

首先，EBGP的Administrative Distance默认是20。而IBGP的Administrative Distance默认是200。

其次，EBGP和IBGP在传递路由的时候，对next-hop的处理不同。**EBGP会修改路由的next-hop，再转发；而IBGP默认不会修改next-hop，直接将路由转发**。举个例子：R4有一条路由要发出，路由传递到R2，R5，路由的下一跳（next-hop）是R4。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/ebgp-vs-ibgp-02.jpg)

R2通过EBGP将路由传输到R1，由于是EBGP，所以路由的next-hop改为R2。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/ebgp-vs-ibgp-03.jpg)

R1再通过EBGP将路由传递给R3，由于是EBGP，路由的next-hop被改为R1。到此为止，R3不用知道R2在哪，只需要将prefix匹配的IP packet发到R1。R1 也不用知道R4在哪，只需要发给R2，R2由于跟R4直连，会最终将请求发送到R4。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/ebgp-vs-ibgp-04.jpg)

接下来，R3再通过IBGP将路由传递给R6，R7。由于是IBGP，默认next-hop不会变化，那么R6，R7收到的路由的next-hop仍然是R1。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/ebgp-vs-ibgp-05.jpg)

这个时候存在问题，由于R6，R7没有与R1直连，也就是说R6，R7不一定能找到R1，相应的路由也有可能被标成无效的。这个时候有两种方法：

- 将R1的地址通过路由协议传到R6，R7
- 在R3连接R6，R7时，设置next-hop-self，也就是修改IBGP的默认行为，让IBGP在传递路由的时候，将路由的next-hop改为自己

第二种方法更为常见，所以在AS的edge router上，通常对IBGP邻居会加上next-hop-self。

EBGP和IBGP在技术实现上的第三个区别在路由转发的行为上。**通过IBGP学习到的路由，不能传递给其他的IBGP。**这么作是为了防止路由环路（loop）。EBGP通过BGP协议里面的AS_PATH和其他元素过滤来自于自己的路由，但是IBGP运行在一个AS内部，没有AS_PATH，所以IBGP干脆不转发来自于其他IBGP的路由。

由于不能转发路由，这要求所有的IBGP router两两相连，组成一个full-mesh的网络。Full-mesh的连接数与节点的关系是n*(n-1)，连接数随着节点数的增加而迅速增加，这给配置和管理带来了问题。

## **IBGP优化**

为了避免full-mesh的连接方式，常见的IBGP优化有两种，一种是Route Reflector，一种是BGP Confederation。

## Route Reflector

Route Reflector，路由反射器，这是一个特殊的IBGP router。一般的IBGP router不会传递来自其他IBGP router的路由。但路由反射器是例外，它会将学习到的IBGP路由，传递给所有连接的RR-client。这样，一个BGP router的路由可以不用与其他的BGP router建立连接，而通过路由反射器发送给其他的BGP router。路由反射器可以大大减少BGP peer的连接数，例如有5个BGP router，那么full-mesh需要管理总共10个BGP peer连接，而使用路由反射器，只需要4个BGP peer。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/ebgp-vs-ibgp-06.jpg)

## BGP Confederation

BGP Confederation是将一个大的AS里面的BGP router，划分到多个小的sub-AS。通过这样的划分，减少IBGP peer连接数。例如，6个BGP router的full-mesh连接数是15。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/ebgp-vs-ibgp-07.jpg)

通过划分sub-AS，最终的连接数降到了8个。

![img](https://github.com/lizj3624/mynote/tree/master/Cloud-Native/pictures/ebgp-vs-ibgp-08.jpg)

以上就是BGP协议的两大类应用：EBGP和IBGP的区别和联系。

