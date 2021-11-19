1、介绍NetFilter和iptables框架

其在内核中的位置如下图:

![1]()

如上图，分三种情况介绍数据包和钩子函数的关系：

![2]()

1.当数据包从物理层和数据链路层传输过来，如果数据包是访问Linux主机本身。则经过PRE_ROUTING和LOCAL_IN钩子函数，到达传输层和应用层。

2.当数据包从物理层和数据链路层传输过来，如果数据包需要转发，则经过PRE_ROUTING、FORWARD和POST_ROUTING三个钩子函数。

3.当数据包从Linux主机本身向外发送数据包，要经过LOCAL_OUT和POST_ROUTING钩子函数。

经过这些钩子函数后，数据包就被捕获了，捕获后处理数据包的规则就在表里面。比如：包过滤的表Filter。

二、介绍iptables的表和链

上图的链和之前图的钩子函数对应
如上图右侧：
Filter表包括三个链：INPUT，OUTPUT，FORWARD，可以在三个位置实现数据包过滤

NAT表包括三个链：PREROUTING，OUTPUT，POSTROUTING，可以在三个位置实现网络地址转换和端口映射

Mangle表包括三个链：INPUT，OUTPUT，FORWARD，PREROUTING，POSTROUTING

添加规则时，我们可以用iptables命令实现。如：

```shell
iptables -t filter -I INPUT #（向Filter表的INPUT链添加一条规则）
```