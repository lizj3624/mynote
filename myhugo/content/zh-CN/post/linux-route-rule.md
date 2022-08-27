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

### 路由表
路由表标记数据包转发的方向，可以通过`ip route show`查看路由表信息
```shell
[root@my ~]$ip route show table local
broadcast 127.0.0.0 dev lo proto kernel scope link src 127.0.0.1 
local 127.0.0.0/8 dev lo proto kernel scope host src 127.0.0.1 
local 127.0.0.1 dev lo proto kernel scope host src 127.0.0.1 
broadcast 127.255.255.255 dev lo proto kernel scope link src 127.0.0.1 
broadcast 192.168.139.0 dev ens33 proto kernel scope link src 192.168.139.152 
local 192.168.139.152 dev ens33 proto kernel scope host src 192.168.139.152 
broadcast 192.168.139.255 dev ens33 proto kernel scope link src 192.168.139.152
```

Linux最多可以支持255张路由表，其中有3张表是内置的：

- 表255 本地路由表（Local table）本地接口地址，广播地址，已及 NAT 地址都放在这个表。该路由表由系统自动维护，管理员不能直接修改。
- 表254 主路由表（Main table）如果没有指明路由所属的表，所有的路由都默认都放在这个表里，一般来说，旧的路由工具（如route）所添加的路由都会加到这个表。一般是普通的路由。
- 表253 默认路由表 （Default table）一般来说默认的路由都放在这张表，但是如果特别指明放的也可以是所有的网关路由。
- 还有一张表 0 是保留的，在文件 /etc/iproute2/rt_tables 可以查看和配置路由表的 TABLE_ID 及路由表名称。

> 也可以通过`route -n`命令查看，但是只能查看main表，比如 `ip route show`命令强大

### 策略路由
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
[root@my ~]# ip rule list
0:  from all lookup local 
32765:  from 192.168.19.0/24 lookup test1 
32766:  from all lookup main 
32767:  from all lookup default
```

### 引用
1. [Linux的策略路由](https://www.ujslxw.com/2020/10/19/44.html)
2. [Linux的策略路由](https://linuxgeeks.github.io/2017/03/17/170119-Linux%E7%9A%84%E7%AD%96%E7%95%A5%E8%B7%AF%E7%94%B1/)
3. [Linux IP 路由背后的原理及其工作原理](https://bbs.huaweicloud.com/blogs/359221)
4. [IP-route管理路由](https://blog.csdn.net/chengxuyuanyonghu/article/details/39558643)
5. [通俗理解IP路由](https://segmentfault.com/a/1190000019363010)
6. [Linux系列—策略路由、ip rule、ip route](https://blog.csdn.net/u012758088/article/details/76255543)
