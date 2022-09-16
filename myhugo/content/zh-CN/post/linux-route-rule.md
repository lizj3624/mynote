---
title: "Linux的路由表和策略路由"
date: 2022-08-27T18:57:15+08:00
tags:
   - linux
   - 网络路由 
   - 策略路由
categories:
   - linux
   - 网络路由 
   - 策略路由
toc: true
---
传统的路由是一个指向目标子网的"指路牌"，任何人来"问路"，路由都会明确指向目标。
传统路由这种"不问来人情况"的处理策略越来越不适合现代计算机网络，举例来说"行人与汽车"走的"路"应该是不同的。
这样策略路由就兴起了，策略路由是近些年来兴起的一个比较新的路由概念。
策略路由可以根据多种不同的策略，决定数据包通过的路径。

## 路由表
路由表标记数据包转发的方向，可以通过`ip route show`查看路由表信息
```shell
### 查看local路由表
[root@my ~]$ip route show table local
broadcast 127.0.0.0 dev lo proto kernel scope link src 127.0.0.1 
local 127.0.0.0/8 dev lo proto kernel scope host src 127.0.0.1 
local 127.0.0.1 dev lo proto kernel scope host src 127.0.0.1 
broadcast 127.255.255.255 dev lo proto kernel scope link src 127.0.0.1 
broadcast 192.168.139.0 dev ens33 proto kernel scope link src 192.168.139.152 
local 192.168.139.152 dev ens33 proto kernel scope host src 192.168.139.152 
broadcast 192.168.139.255 dev ens33 proto kernel scope link src 192.168.139.152

### 查看路由表 
ip route list

### 精准查看具体某一条路由
ip route show [exact] 169.254.0.0/16

### 模糊匹配某一条路由
ip route show match 172.18

### 仅列出源地址前缀为172.18.16.0/20的路由
ip route show src 172.18.16.0/20

### 仅列出通过前缀选择的为该ip的路由
ip route show via 172.18.31.253

### 获取到目标的单个路由，并按照内核所看到的方式打印其内容
ip route get 169.254.0.0/16

### 删除192.168.4.0网段的网关
ip route del 192.168.4.0/24

### 删除默认路由
ip route del default

### 删除路由
ip route delete 192.168.1.0/24 dev eth0

### 将路由表信息保存到标准输出
ip route save

### 从stdin恢复路由表信息 该命令希望读取从ip route save返回的数据流
ip route restore

### 该flush选项与ip route一起使用时，将清空路由表或删除特定目标的路由
### 删除特定路由
ip route flush 10.38.0.0/16

ip route flush table main
```

Linux最多可以支持255张路由表，其中有3张表是内置的：

- 表255 本地路由表（Local table）本地接口地址，广播地址，已及 NAT 地址都放在这个表。该路由表由系统自动维护，管理员不能直接修改。
- 表254 主路由表（Main table）如果没有指明路由所属的表，所有的路由都默认都放在这个表里，一般来说，旧的路由工具（如route）所添加的路由都会加到这个表。一般是普通的路由。
- 表253 默认路由表 （Default table）一般来说默认的路由都放在这张表，但是如果特别指明放的也可以是所有的网关路由。
- 还有一张表 0 是保留的，在文件 /etc/iproute2/rt_tables 可以查看和配置路由表的 TABLE_ID 及路由表名称。

### 使用route命令分析路由
```shell
# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.139.128.1    0.0.0.0         UG    0      0        0 eth0
10.0.0.10       10.139.128.1    255.255.255.255 UGH   0      0        0 eth0
10.139.128.0    0.0.0.0         255.255.224.0   U     0      0        0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.18.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-0ab63c131848
172.19.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-bccbfb788da0
172.20.0.0      0.0.0.0         255.255.0.0     U     0      0        0 br-7485db25f958
```
- `Destination`: 目标网络或目标主机。Destination为default（0.0.0.0）时，表示这个是默认网关，所有数据都发到这个网关（这里是 10.139.128.1）
- `Gateway`: 网关地址，0.0.0.0 表示当前记录对应的 Destination 跟本机在同一个网段，通信时不需要经过网关
- `Genmask`: Destination字段的网络掩码，Destination是主机时需要设置为255.255.255.255，是默认路由时会设置为0.0.0.0
- `Flags`: 标记
   * U 路由是活动的
   * H 目标是个主机
   * G 需要经过网关
   * R 恢复动态路由产生的表项
   * D 由路由的后台程序动态地安装
   * M 由路由的后台程序修改
   * ! 拒绝路由
- `Metric`: 路由距离，到达指定网络所需的中转数，是大型局域网和广域网设置所必需的 （不在Linux内核中使用）
- `Ref`: 路由项引用次数 （不在Linux内核中使用）
- `Use`: 此路由项被路由软件查找的次数
- `Iface`: 网卡名字，例如eth0

### Linux 内核的路由种类
#### 主机路由
路由表中指向单个 IP 地址或主机名的路由记录，其 Flags 字段为 H。下面示例中，对于 10.0.0.10 这个主机，通过网关 10.139.128.1 网关路由：
```shell
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.10       10.139.128.1    255.255.255.255 UGH   0      0        0 eth0
...
```
#### 网络路由
主机可以到达的网络。下面示例中，对于 10.0.0.0/24 这个网络，通过网关 10.139.128.1 网关路由：
```shell
[root@VM_139_74_centos ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.0.0.0        10.139.128.1    255.255.255.0   UG    0      0        0 eth0
```
#### 默认路由
当目标主机的 IP 地址或网络不在路由表中时，数据包就被发送到默认路由（默认网关）上。默认路由的 Destination 是 default 或 0.0.0.0。
```shell
[root@VM_139_74_centos ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    0      0        0 eth0
```

> 可以通过`route -n`命令查看，但是只能查看main表，比如 `ip route show`命令强大

## 策略路由
策略是指对于IP包的路由是以我们根据需要而定下的一些策略为主要依据进行路由的。
例如我们可以有这样的策略："所有来直自网A的包，选择X路径；其他选择Y路径"，或者是“所有TOS为A的包选择路径F；其他选者路径K”。

Linux是通过规则(rule)来支持策略路由的

我们可以用自然语言这样描述规则：
- 规则一："所有来自192.16.152.24的IP包，使用路由表10，本规则的优先级是990"
- 规则三："所有到192.168.127.127的IP包，使用路由表11，本规则的优先级是991"
- 规则二："所有的包，使用路由表253，本规则的优先级是32767"

我们可以看到，规则包含3个要素：
- 什么样的包，将应用本规则（所谓的SELECTOR，可能是filter更能反映其作用）；
- 符合本规则的包将对其采取什么动作（ACTION），例如：使用哪个路由个表；
- 本规则的优先级。优先级别越高的规则越先匹配（数值越小优先级别越高）。

可以通过`ip rule list`查看规则
```shell
### 查看策略路由(路由规则)
# ip rule list
0:  from all lookup local 
32765:  from 192.168.19.0/24 lookup test1 
32763:	from 114.13.28.0/24 lookup 100 
32766:  from all lookup main 
32767:  from all lookup default

### 查看策略路由引用的路由表ip route show table table_name/table_number
### 查看100路由表信息
# ip route show table 100
default via 111.120.35.254 dev eth0.450
```

## 引用
1. [Linux的策略路由](https://www.ujslxw.com/2020/10/19/44.html)
2. [Linux的策略路由](https://linuxgeeks.github.io/2017/03/17/170119-Linux%E7%9A%84%E7%AD%96%E7%95%A5%E8%B7%AF%E7%94%B1/)
3. [Linux IP 路由背后的原理及其工作原理](https://bbs.huaweicloud.com/blogs/359221)
4. [IP-route管理路由](https://blog.csdn.net/chengxuyuanyonghu/article/details/39558643)
5. [通俗理解IP路由](https://segmentfault.com/a/1190000019363010)
6. [Linux系列—策略路由、ip rule、ip route](https://blog.csdn.net/u012758088/article/details/76255543)
