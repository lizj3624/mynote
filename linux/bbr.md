# 什么是BBR：
TCP BBR是谷歌出品的TCP拥塞控制算法。BBR目的是要尽量跑满带宽，并且尽量不要有排队的情况。BBR可以起到单边加速TCP连接的效果。

Google提交到Linux主线并发表在ACM queue期刊上的TCP-BBR拥塞控制算法。继承了Google“先在生产环境上部署，再开源和发论文”的研究传统。TCP-BBR已经再YouTube服务器和Google跨数据中心的内部广域网(B4)上部署。由此可见出该算法的前途。

TCP-BBR的目标就是最大化利用网络上瓶颈链路的带宽。一条网络链路就像一条水管，要想最大化利用这条水管，最好的办法就是给这跟水管灌满水。

BBR解决了两个问题：

在有一定丢包率的网络链路上充分利用带宽。非常适合高延迟，高带宽的网络链路。

降低网络链路上的buffer占用率，从而降低延迟。非常适合慢速接入网络的用户。
Google 在 2016年9月份开源了他们的优化网络拥堵算法BBR，最新版本的 Linux内核(4.9-rc8)中已经集成了该算法。

对于TCP单边加速，并非所有人都很熟悉，不过有另外一个大名鼎鼎的商业软件“锐速”，相信很多人都清楚。特别是对于使用国外服务器或者VPS的人来说，效果更佳。

BBR项目地址： https://github.com/google/bbr

# 安装BBR

## 安装4.9以上的内核

## 配置BBR
```shell
#查看内核版本
[root@amber ~]# uname -r
4.18.5-1.el7.elrepo.x86_64

# 开启bbr
vim /etc/sysctl.conf
# 在文件末尾添加如下内容
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr

sysctl -p

# 确定bbr已经成功开启
[root@amber ~]# sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = reno cubic bbr
[root@amber ~]# lsmod | grep bbr
tcp_bbr                20480  1
```


