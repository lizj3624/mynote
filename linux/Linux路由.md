# Linux 内核的路由表

通过 `route` 命令查看 Linux 内核的路由表：

```bash
[root@VM_139_74_centos ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    0      0        0 eth0
10.0.0.10       10.139.128.1    255.255.255.255 UGH   0      0        0 eth0
10.139.128.0    0.0.0.0         255.255.224.0   U     0      0        0 eth0
link-local      0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-0ab63c131848
172.19.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-bccbfb788da0
172.20.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-7485db25f958
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.139.128.1    0.0.0.0         UG    0      0        0 eth0
10.0.0.10       10.139.128.1    255.255.255.255 UGH   0      0        0 eth0
10.139.128.0    0.0.0.0         255.255.224.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-0ab63c131848
172.19.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-bccbfb788da0
172.20.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-7485db25f95812345678910111213141516171819202122
```

各列字段说明：

| 列          | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| Destination | 目标网络或目标主机。Destination 为 default（`0.0.0.0`）时，表示这个是默认网关，所有数据都发到这个网关（这里是 `10.139.128.1`） |
| Gateway     | 网关地址，`0.0.0.0` 表示当前记录对应的 Destination 跟本机在同一个网段，通信时不需要经过网关 |
| Genmask     | Destination 字段的网络掩码，Destination 是主机时需要设置为 `255.255.255.255`，是默认路由时会设置为 `0.0.0.0` |
| Flags       | 标记，含义参考表格后面的解释                                 |
| Metric      | 路由距离，到达指定网络所需的中转数，是大型局域网和广域网设置所必需的 （不在Linux内核中使用。） |
| Ref         | 路由项引用次数 （不在Linux内核中使用。）                     |
| Use         | 此路由项被路由软件查找的次数                                 |
| Iface       | 网卡名字，例如 `eth0`                                        |

Flags 含义：

- U 路由是活动的
- H 目标是个主机
- G 需要经过网关
- R 恢复动态路由产生的表项
- D 由路由的后台程序动态地安装
- M 由路由的后台程序修改
- ! 拒绝路由

## Linux 内核的路由种类

### 主机路由

路由表中指向单个 IP 地址或主机名的路由记录，其 Flags 字段为 H。下面示例中，对于 `10.0.0.10` 这个主机，通过网关 `10.139.128.1` 网关路由：

```bash
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.10       10.139.128.1    255.255.255.255 UGH   0      0        0 eth0
...12345
```

### 网络路由

主机可以到达的网络。下面示例中，对于 `10.0.0.0/24` 这个网络，通过网关 `10.139.128.1` 网关路由：

```shell
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.0        10.139.128.1    255.255.255.0   UG    0      0        0 eth01234
```

### 默认路由

当目标主机的 IP 地址或网络不在路由表中时，数据包就被发送到默认路由（默认网关）上。默认路由的 `Destination` 是 default 或 `0.0.0.0`。

```bash
[root@VM_139_74_centos ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    0      0        0 eth01234
```

## route 命令

route 命令可以显示或设置 Linux 内核中的路由表，主要是静态路由。

对于局域网中的 Linux 主机，要想访问 Internet，需要将局域网的网关 IP 地址设置为这个主机的默认路由。在命令行中通过 `route` 命令添加的路由在网卡重启或机器重启后失效。可以在 `/etc/rc.local` 中添加 route 命令来保证路由设置永久有效。

选项:

- `-A`：设置地址类型
- `-C`：打印 Linux 内核的路由缓存
- `-v`：显示详细信息
- `-n`：不执行 DNS 反向查找，直接显示数字形式的 IP 地址
- `-e`：netstat 格式显示路由表
- `-net`：到一个网络的路由表
- `-host`：到一个主机的路由表

参数：

- add：增加路由记录
- del：删除路由记录
- target：目的网络或目的主机
- gw：设置默认网关
- mss：设置TCP的最大区块长度（MSS），单位MB
- window：指定通过路由表的TCP连接的TCP窗口大小
- dev：路由记录所表示的网络接口

### 添加路由 add

可以添加一条可用路由，或添加一条要屏蔽的路由。

#### 添加主机路由

添加主机路由时，需要指定网络 ID 和主机 ID，此时需要设置 `netmask 255.255.255.255`：

```bash
[root@VM_139_74_centos ~]# route add -net 10.0.0.10 netmask 255.255.255.255 gw 10.139.128.1 dev eth0
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.10       10.139.128.1    255.255.255.255 UGH   0      0        0 eth0
...123456
```

#### 添加网络路由

添加网络路由时，只需指定网络 ID，通过 `netmask` 设置掩码长度：

```bash
[root@VM_139_74_centos ~]# route add -net 10.0.0.0 netmask 255.255.255.0 gw 10.139.128.1 dev eth0
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.0        10.139.128.1    255.255.255.0   UG    0      0        0 eth0
...123456
```

#### 添加添加同一个局域网的主机

不指定 gw 选项时，添加的路由记录不使用网关：

```bash
[root@VM_139_74_centos ~]# route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
224.0.0.0       0.0.0.0         240.0.0.0       U     0      0        0 eth0
...123456
```

### 屏蔽路由

```shell
[root@VM_139_74_centos ~]# route add -net 224.0.0.0 netmask 240.0.0.0 reject
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
224.0.0.0       -               240.0.0.0       !     0      -        0 -
...123456
```

### 删除路由记录

跟添加路由类似，可以删除一条可用路由，或删除一条屏蔽的路由。

#### 删除可用路由

```
route del -net 224.0.0.0 netmask 240.0.0.01
```

#### 删除屏蔽的路由

```
route del -net 224.0.0.0 netmask 240.0.0.0 reject1
```

#### 删除和添加设置默认网关

添加或删除默认网关时，Linux 会自动检查网关的可用性：

```bash
[root@VM_139_74_centos ~]# route add default gw 192.168.1.1
SIOCADDRT: Network is unreachable
[root@VM_139_74_centos ~]# route del default gw 192.168.1.1
SIOCDELRT: No such process
```

## 引用

1. [Linux路由表详解及route命令详解](https://blog.csdn.net/kikajack/article/details/80457841)

2. [route命令](https://man.linuxde.net/route)



# Linux traceroute命令

traceroute 显示数据包到主机间的路径

## traceroute命令

**traceroute命令** 用于追踪数据包在网络上的传输时的全部路径，它默认发送的数据包大小是40字节。

通过traceroute我们可以知道信息从你的计算机到互联网另一端的主机是走的什么路径。当然每次数据包由某一同样的出发点（source）到达某一同样的目的地(destination)走的路径可能会不一样，但基本上来说大部分时候所走的路由是相同的。

traceroute通过发送小的数据包到目的设备直到其返回，来测量其需要多长时间。一条路径上的每个设备traceroute要测3次。输出结果中包括每次测试的时间(ms)和设备的名称（如有的话）及其ip地址。

### 语法

```shell
traceroute(选项)(参数)
```

### 选项

```shell
-d：使用Socket层级的排错功能；
-f<存活数值>：设置第一个检测数据包的存活数值TTL的大小；
-F：设置勿离断位；
-g<网关>：设置来源路由网关，最多可设置8个；
-i<网络界面>：使用指定的网络界面送出数据包；
-I：使用ICMP回应取代UDP资料信息；
-m<存活数值>：设置检测数据包的最大存活数值TTL的大小；
-n：直接使用IP地址而非主机名称；
-p<通信端口>：设置UDP传输协议的通信端口；
-r：忽略普通的Routing Table，直接将数据包送到远端主机上。
-s<来源地址>：设置本地主机送出数据包的IP地址；
-t<服务类型>：设置检测数据包的TOS数值；
-v：详细显示指令的执行过程；
-w<超时秒数>：设置等待远端主机回报的时间；
-x：开启或关闭数据包的正确性检验。
```

### 参数

主机：指定目的主机IP地址或主机名。

### 实例

```shell
traceroute www.58.com
traceroute to www.58.com (211.151.111.30), 30 hops max, 40 byte packets
 1  unknown (192.168.2.1)  3.453 ms  3.801 ms  3.937 ms
 2  221.6.45.33 (221.6.45.33)  7.768 ms  7.816 ms  7.840 ms
 3  221.6.0.233 (221.6.0.233)  13.784 ms  13.827 ms 221.6.9.81 (221.6.9.81)  9.758 ms
 4  221.6.2.169 (221.6.2.169)  11.777 ms 122.96.66.13 (122.96.66.13)  34.952 ms 221.6.2.53 (221.6.2.53)  41.372 ms
 5  219.158.96.149 (219.158.96.149)  39.167 ms  39.210 ms  39.238 ms
 6  123.126.0.194 (123.126.0.194)  37.270 ms 123.126.0.66 (123.126.0.66)  37.163 ms  37.441 ms
 7  124.65.57.26 (124.65.57.26)  42.787 ms  42.799 ms  42.809 ms
 8  61.148.146.210 (61.148.146.210)  30.176 ms 61.148.154.98 (61.148.154.98)  32.613 ms  32.675 ms
 9  202.106.42.102 (202.106.42.102)  44.563 ms  44.600 ms  44.627 ms
10  210.77.139.150 (210.77.139.150)  53.302 ms  53.233 ms  53.032 ms
11  211.151.104.6 (211.151.104.6)  39.585 ms  39.502 ms  39.598 ms
12  211.151.111.30 (211.151.111.30)  35.161 ms  35.938 ms  36.005 ms
```

记录按序列号从1开始，每个纪录就是一跳 ，每跳表示一个网关，我们看到每行有三个时间，单位是ms，其实就是`-q`的默认参数。探测数据包向每个网关发送三个数据包后，网关响应后返回的时间；如果用`traceroute -q 4 www.58.com`，表示向每个网关发送4个数据包。

有时我们traceroute一台主机时，会看到有一些行是以星号表示的。出现这样的情况，可能是防火墙封掉了ICMP的返回信息，所以我们得不到什么相关的数据包返回数据。

有时我们在某一网关处延时比较长，有可能是某台网关比较阻塞，也可能是物理设备本身的原因。当然如果某台DNS出现问题时，不能解析主机名、域名时，也会 有延时长的现象；您可以加`-n`参数来避免DNS解析，以IP格式输出数据。

如果在局域网中的不同网段之间，我们可以通过traceroute 来排查问题所在，是主机的问题还是网关的问题。如果我们通过远程来访问某台服务器遇到问题时，我们用到traceroute 追踪数据包所经过的网关，提交IDC服务商，也有助于解决问题；但目前看来在国内解决这样的问题是比较困难的，就是我们发现问题所在，IDC服务商也不可能帮助我们解决。

**跳数设置**

```shell
[root@localhost ~]# traceroute -m 10 www.baidu.com
traceroute to www.baidu.com (61.135.169.105), 10 hops max, 40 byte packets
 1  192.168.74.2 (192.168.74.2)  1.534 ms  1.775 ms  1.961 ms
 2  211.151.56.1 (211.151.56.1)  0.508 ms  0.514 ms  0.507 ms
 3  211.151.227.206 (211.151.227.206)  0.571 ms  0.558 ms  0.550 ms
 4  210.77.139.145 (210.77.139.145)  0.708 ms  0.729 ms  0.785 ms
 5  202.106.42.101 (202.106.42.101)  7.978 ms  8.155 ms  8.311 ms
 6  bt-228-037.bta.net.cn (202.106.228.37)  772.460 ms bt-228-025.bta.net.cn (202.106.228.25)  2.152 ms 61.148.154.97 (61.148.154.97)  772.107 ms
 7  124.65.58.221 (124.65.58.221)  4.875 ms 61.148.146.29 (61.148.146.29)  2.124 ms 124.65.58.221 (124.65.58.221)  4.854 ms
 8  123.126.6.198 (123.126.6.198)  2.944 ms 61.148.156.6 (61.148.156.6)  3.505 ms 123.126.6.198 (123.126.6.198)  2.885 ms
 9  * * *
10  * * *
```

其它一些实例

```shell
traceroute -m 10 www.baidu.com # 跳数设置
traceroute -n www.baidu.com    # 显示IP地址，不查主机名
traceroute -p 6888 www.baidu.com  # 探测包使用的基本UDP端口设置6888
traceroute -q 4 www.baidu.com  # 把探测包的个数设置为值4
traceroute -r www.baidu.com    # 绕过正常的路由表，直接发送到网络相连的主机
traceroute -w 3 www.baidu.com  # 把对外发探测包的等待响应时间设置为3秒
```