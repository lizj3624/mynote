### 一、介绍NetFilter和iptables框架

其在内核中的位置如下图:

![1](https://github.com/lizj3624/mynote/blob/master/Linux/pictures/iptables.png)

如上图，分三种情况介绍数据包和钩子函数的关系：

![2](https://github.com/lizj3624/mynote/blob/master/Linux/pictures/netfilter.png)

1.当数据包从物理层和数据链路层传输过来，如果数据包是访问Linux主机本身。则经过`PRE_ROUTING`和`LOCAL_IN`钩子函数，到达传输层和应用层。

2.当数据包从物理层和数据链路层传输过来，如果数据包需要转发，则经过`PRE_ROUTING、FORWARD和POST_ROUTING`三个钩子函数。

3.当数据包从Linux主机本身向外发送数据包，要经过`LOCAL_OUT`和`POST_ROUTING`钩子函数。

经过这些钩子函数后，数据包就被捕获了，捕获后处理数据包的规则就在表里面。比如：包过滤的表Filter。

### 二、介绍iptables的表和链

上图的链和之前图的钩子函数对应
如上图右侧：

![3](https://github.com/lizj3624/mynote/blob/master/Linux/pictures/iptable-hook.png)

Filter表包括三个链：`INPUT，OUTPUT，FORWARD`，可以在三个位置实现数据包过滤

NAT表包括三个链：`PREROUTING，OUTPUT，POSTROUTING`，可以在三个位置实现网络地址转换和端口映射

Mangle表包括三个链：`INPUT，OUTPUT，FORWARD，PREROUTING，POSTROUTING`

添加规则时，我们可以用iptables命令实现。如：

```shell
iptables -t filter -I INPUT #（向Filter表的INPUT链添加一条规则）
```

其主要通过表、链实现规则，可以这么说，Netfilter是表的容器，表是链的容器，链是规则的容器，最终形成对数据报处理规则的实现。简单地讲， tables 由 chains 组成，而 chains 又由 rules 组成。 iptables 默认有四个表 Filter, NAT, Mangle, Raw。

优先级顺序是：raw —> mangle —> nat —> filter。也就是说在某一个链上有多张表，数据包都会依次按照hook点的方向进行传输，每个hook点上Netfilter又按照优先级挂了很多hook函数（即表），就是按照这个顺序依次处理。无论那一个Filter表其匹配原则都是“First Match”，即优先执行，第一条规则逐一向下匹配，如果封包进来遇到第一条规则允许通过，那么这个封包就通过，而不管下面的rule2、rule3的规则是什么都不重要；相反如果第一条规则说要丢弃，即便是rule2规则允许通过也不起任何作用，这就是“first match”原则。

### 三、Iptables基本操作

#### 配置防火墙

```shell
#先检查是否安装了iptables
service iptables status
#安装iptables
yum install -y iptables
#升级iptables
yum update iptables
#安装iptables-services
yum install iptables-services

#禁用/停止自带的firewalld服务
#停止firewalld服务
systemctl stop firewalld
#禁用firewalld服务
systemctl mask firewalld
```

#### service命令

```shell
# 启动 iptables 
service iptables start

# 关闭iptables
service iptables stop

# 重启iptables
service iptables restart

# 查看iptables状态： 
service iptables status

# 保存iptables配置 
service iptables save
```

#### systemctl命令

```shell
# 启动防火墙
systemctl start iptables.service

# 停止防火墙
systemctl stop iptables.service

# 重启防火墙，才能使规则生效
systemctl restart iptables.service

# 设置防火墙开机启动
systemctl enable iptables.service

#防火墙开机禁用
systemctl disable iptables.service

# 查看状态
systemctl status iptables.service
```

#### 其它

```shell
# Iptables服务配置文件
/etc/sysconfig/iptables-config

# Iptables` 规则保存文件
/etc/sysconfig/iptables

# 配置的规则进行存档
service iptables save

# 保存当前iptables规则到配置文件 
iptables-save > /etc/sysconfig/iptables

# 从配置文件，恢复iptables规则
iptables-restore < /etc/sysconfig/iptables

# 打开iptables转发
echo "1"> /proc/sys/net/ipv4/ip_forward

# 查看当前系统中生效的所有参数
/usr/sbin/sysctl –a
```



### Iptables语法以常见命令

`Iptables [-t table] command [chain] [rules] [-j target]`

* 表名(table)： `Filter, NAT, Mangle, Raw` 起包过滤功能的为表 Filter ，可以不填，不填默认为 Filter。

* command部分是iptables命令最重要的部分，它告诉iptables命令要做什么，例如插入规则、将规则添加到链的末尾或删除规则。

* chain 可以根据数据流向来确定具体使用哪个链，在 Filter 中的使用情况如下：

    > INPUT链 – 处理来自外部的数据。
    > OUTPUT链 – 处理向外发送的数据。
    > FORWARD链 – 将数据转发到本机的其他网卡设备上。

* rules 条件匹配分为基本匹配和扩展匹配，拓展匹配又分为隐式扩展和显示扩展。

* target 数据包控制方式包括四种为
> ACCEPT：允许数据包通过
> DROP：直接丢弃数据包，不给出任何回应信息
> REJECT: 拒绝，使用–reject-with选项可以提示信息，有以下可用值  
> SNAT：源地址转换

#### Iptables常见命令
```shell

# 统计服务器上的IP：192.168.0.10的入网流量：
iptables -I INPUT -d 192.168.0.10`

# 统计该IP的出网流量：
iptables -I OUTPUT -s 192.168.0.10`

# 默认是使用易读的单位，也就是自动转化成M，G。如过需要Bytes做单位，则增加一个-x参数
iptables -n -v -L -t filter  

iptables -n -v -L -t filter -x

# 显示每条规则的序列号和额外信息
iptables -L -n -v --line-number

# 在几号规则前插入一条所有都能通过规则
iptables -I FORWARD 1 -j ACCEPT
```

### nf_conntrack模块

nf_conntrack(在老版本的 Linux 内核中叫 ip_conntrack)是一个内核模块,用于跟踪一个连接的状态的。连接状态跟踪可以供其他模块使用,最常见的两个使用场景是 iptables 的 nat 的 state 模块。 iptables 的 nat 通过规则来修改目的/源地址,但光修改地址不行,我们还需要能让回来的包能路由到最初的来源主机。这就需要借助 nf_conntrack 来找到原来那个连接的记录才行。而 state 模块则是直接使用 nf_conntrack 里记录的连接的状态来匹配用户定义的相关规则。例如下面这条 INPUT 规则用于放行 80 端口上的状态为 NEW 的连接上的包。

#### nf_conntrack模块常用命令

```shell
# 查看nf_conntrack表当前连接数    
cat /proc/sys/net/netfilter/nf_conntrack_count       

# 查看nf_conntrack表最大连接数    
cat /proc/sys/net/netfilter/nf_conntrack_max    

# 通过dmesg可以查看nf_conntrack的状况：
dmesg |grep nf_conntrack

# 查看存储conntrack条目的哈希表大小,此为只读文件
cat /proc/sys/net/netfilter/nf_conntrack_buckets

# 查看nf_conntrack的TCP连接记录时间
cat /proc/sys/net/netfilter/nf_conntrack_tcp_timeout_established

# 通过内核参数查看命令，查看所有参数配置
sysctl -a | grep nf_conntrack

# 通过conntrack命令行工具查看conntrack的内容
yum install -y conntrack  
conntrack -L  

# 加载对应跟踪模块
[root@plop ~]# modprobe /proc/net/nf_conntrack_ipv4    
[root@plop ~]# lsmod | grep nf_conntrack    
nf_conntrack_ipv4       9506  0    
nf_defrag_ipv4          1483  1 nf_conntrack_ipv4    
nf_conntrack_ipv6       8748  2    
nf_defrag_ipv6         11182  1 nf_conntrack_ipv6    
nf_conntrack           79758  3 nf_conntrack_ipv4,nf_conntrack_ipv6,xt_state    
ipv6                  317340  28 sctp,ip6t_REJECT,nf_conntrack_ipv6,nf_defrag_ipv6  

# 移除 nf_conntrack 模块
$ sudo modprobe -r xt_NOTRACK nf_conntrack_netbios_ns nf_conntrack_ipv4 xt_state
$ sudo modprobe -r nf_conntrack

# 查看当前的连接数:
grep nf_conntrack /proc/slabinfo

# 查出目前 nf_conntrack 的排名:
cat /proc/net/nf_conntrack | cut -d ' ' -f 10 | cut -d '=' -f 2 | sort | uniq -c | sort -nr | head -n 10
```

#### 会话表满的解决办法

错误日志信息

```shell
less /var/log/messages
Nov  3 23:30:27 digoal_host kernel: : [63500383.870591] nf_conntrack: table full, dropping packet.  
Nov  3 23:30:27 digoal_host kernel: : [63500383.962423] nf_conntrack: table full, dropping packet.  
Nov  3 23:30:27 digoal_host kernel: : [63500384.060399] nf_conntrack: table full, dropping packet.  
```

**解决方法举例**：

1、排查是否DDoS攻击，如果是，从预防攻击层面解决问题。

2、清空会话表。

重启iptables，会自动清空nf_conntrack table。注意，重启前先保存当前iptables配置(iptables-save > /etc/sysconfig/iptables ; service iptables restart)。

3、应用程序正常关闭会话

设计应用时，正常关闭会话很重要。

4、加大表的上限（需要考虑内存的消耗）

#### 重要的几个配置文件

`nf_conntrack_max`决定连接跟踪表的大小,当nf_conntrack模块被装置且服务器上连接超过这个设定的值时，系统会主动丢掉新连接包，直到连接小于此设置值才会恢复。
`nf_conntrack_buckets`决定存储conntrack条目的哈希表大小，若是单方面修改`nf_conntrack_max`，而不修改`nf_conntrack_buckets`，只是影响查找速度，挂在不了桶上的新跟踪项目，会挂在到桶中的链表上（原理为hash表结构）。
`nf_conntrack_tcp_timeout_established`系统默认值为”432000”，代表nf_conntrack的TCP连接记录时间默认是5天，致使nf_conntrack的值减不下来，丢包持续时间长。
通过修改这两个值即可，但是**nf_conntrack_buckets时个只读文件**，无法进行修改。

#### **修改参数**

- 或通过sysctl命令进行修改：

    ```shell
    $ sysctl -w net.netfilter.nf_conntrack_max=1048576
    $ sysctl -w net.netfilter.nf_conntrack_tcp_timeout_established=3600
    $ sysctl -p #使生效
    ```

- 或是直接永久性修改永久生效

    ```shell
    vi /etc/sysctl.conf  
    net.netfilter.nf_conntrack_max=1048576
    net.netfilter.nf_conntrack_tcp_timeout_established=3600         
    ```


> 1. [netfilter和iptables的实现机制](https://blog.csdn.net/wxywxywxy110/article/details/78621789)
> 2. [Iptables汇总](https://clodfisher.github.io/2018/08/IptablesCollect/)
> 3. [Iptables之nf_conntrack模块](https://clodfisher.github.io/2018/09/nf_conntrack/)
