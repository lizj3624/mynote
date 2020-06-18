## Linux CPU监控指令

### Linux衡量CPU性能的指标：

* 用户使用CPU：实时运行的进程、nice
* 系统使用CPU：用于I/O管理的中断和驱动、用于内存管理的页面交换、用于进程管理的进程开始和上下文切换
* WIO：用于进程等待磁盘I/O而使CPU处于空闲状态的比率
* CPU的空闲率，除了上面的WIO以外的空闲时间
* CPU用于上下文交换的比率
* nice
* Real-time
* 运行进程队列的长度
* 平均负载：load

###  Linux常用监控CPU整体性能的工具
![monitor](https://github.com/lizj3624/mynote/blob/master/Linux-Performance/pictures/linux-cpu-monitor.png)

#### top

**top**，观察系统整体CPU使用率、平均负载、软硬中断等性能指标，其中wa指标有明显变化。

#### mpstat

mpstat命令：主要用于多CPU环境下，它显示各个可用CPU的状态。这些信息存放在/proc/stat文件中。在多CPUs系统里，其不但能查看所有CPU的平均状况信息，而且能够查看特定CPU的信息。

```shell
##命令格式:
mpstat [ 选项 ] [ <时间间隔> [ <次数> ] ]

##选项说明：
[ -A ] 展示所有cpu的状态
[ -I { SUM | CPU | ALL } ] 分三种情况展示 cpu的状态
[ -u ]
[ -P { [,…] | ON | ALL } ] 指定某个cpu，进行展示
[ -V ] 版本信息

## 常用命令
mpstat -u

mpstat -P 1

mpstat -A

mpstat -P ALL 3 5  ##每隔3s采集所有CPU数据，采集5次，然后汇总
11时02分12秒  CPU    %usr   %nice    %sys %iowait    %irq   %soft  %steal  %guest   %idle
11时02分15秒  all    9.59    0.00    0.39    0.09    0.00    0.01    0.00    0.00   89.92
11时02分15秒    0   22.59    0.00    0.66    0.33    0.00    0.00    0.00    0.00   76.41
11时02分15秒    1   14.05    0.00    0.67    0.33    0.00    0.00    0.00    0.00   84.95
```

mpstat输出数据说明：

1.CPU （处理器编号，all表示所有处理器的平均数值）

2.%user （用户态的CPU利用率百分比）

3.%nice （用户态的优先级别CPU的利用率百分比）

4.%sys （内核态的CPU利用率百分比）

5.%iowait （在interval间段内io的等待百分比，interval 为采样频率，如本文的1为每一秒钟采样一次） 

6.%irq （在interval间段内,CPU的中断百分比）

7.%soft （在interval间段内,CPU的软中断百分比） 

8.%steal Show the percentage of time spent in involuntary wait by the virtual CPU or CPUs while the hypervisor was servicing another virtual processor.

9.%guest Show the percentage of time spent by the CPU or CPUs to run a virtual processor.

10.%idle （在interval间段内，CPU的闲置百分比，不包括I/O请求的等待）

#### vmstat

vmstat是一个很全面的性能分析工具，可以观察到系统的进程状态、内存使用、虚拟内存使用、磁盘的IO、中断、上下文切换、CPU使用等。对于 Linux 的性能分析，100%理解 vmstat 输出内容的含义，并能灵活应用，那对系统性能分析的能力就算是基本掌握了。

```shell
### 命令格式:
vmstat [-V] [-n] （选项） [delay [count]]（参数）

### 选项说明：
-a：显示活动内页； 
-f：显示启动后创建的进程总数； 
-m：显示slab信息； 
-n：头信息仅显示一次； 
-s：以表格方式显示事件计数器和内存状态； 
-d：报告磁盘状态； 
-p：显示指定的硬盘分区状态； 
-S：输出信息的单位。

### 参数说明：
事件间隔delay：状态信息刷新的时间间隔；
次数count：显示报告的次数。

### 常用命令
vmstat -n 3 3 (三秒一次，共三次)

vmstat -S M 3 2 (以M为单位显示，默认为k)

vmstat -f

vmstat 1 5
procs -----------memory---------- ---swap-- -----io---- --system-- -----cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 8  0      0 17588556  17496 227236    0    0    18     6    7    2 21  1 78  0  0
 8  0      0 17590504  17504 227272    0    0     0    72 16841 26561 26  0 74  0  0
 8  0      0 17588928  17504 227296    0    0     0    24 16722 24820 25  0 74  0  0
 8  0      0 17580056  17520 227272    0    0     0   136 17289 25094 26  1 74  0  0
 8  0      0 17576280  17528 227284    0    0     0   340 14891 22463 25  0 75  0  0
```

vmstat输出数据说明：

1. procs 

    `r`列表示运行和等待CPU时间片的进程数，这个值如果长期大于系统CPU个数，就说明CPU资源不足，可以考虑增加CPU；

​    `b`列表示在等待资源的进程数，比如正在等待I/O或者内存交换等。

2. memory

   `swp`列表示切换到内存交换区的内存数量（以KB为单位）。如果swp的值不为0或者比较大，而且`si、so`的值长期为0，那么这种情况一般不用担心，不会影响系统性能；

   `free`列表示当前空闲的物理内存数量（以KB为单位）；

   `buff`列表示buffers cache的内存数量，一般对块设备的读写才需要缓冲；

   `cache`列表示page cached的内存数量，一般作文件系统的cached，频繁访问的文件都会被cached。如果cached值较大，就说明cached文件数较多。如果此时IO中的bi比较小，就说明文件系统效率比较好。

3. swap

  `si`列表示由磁盘调入内存 ，也就是内存进入内存交换区的数量；

  `so`列表示由内存调入磁盘 ，也就是内存交换区进入内存的数量

  一般情况下，si、so的值都为0，如果si、so的值长期不为0，则表示系统内存不足，需要考虑是否增加系统内存 。

4. IO

  `bi`列表示从块设备读入的数据总量（即读磁盘，单位KB/秒）

  `bo`列表示写入到块设备的数据总量（即写磁盘，单位KB/秒） 这里设置的bi+bo参考值为1000，如果超过1000，而且wa值比较大，则表示系统磁盘IO性能瓶颈。

5. system

  `in`列表示在某一时间间隔中观察到的每秒设备中断数；

  `cs`列表示每秒产生的上下文切换次数。 上面这两个值越大，会看到内核消耗的CPU时间就越多。

6. CPU

  `us`列显示了用户进程消耗CPU的时间百分比。us的值比较高时，说明用户进程消耗的CPU时间多，如果长期大于50%，需要考虑优化程序啥的。

  `sy`列显示了内核进程消耗CPU的时间百分比。sy的值比较高时，就说明内核消耗的CPU时间多；如果us+sy超过80%，就说明CPU的资源存在不足。

  `id`列显示了CPU处在空闲状态的时间百分比；

  `wa`列表示IO等待所占的CPU时间百分比。wa值越高，说明IO等待越严重。如果wa值超过20%，说明IO等待严重 。

  `st`列一般不关注，虚拟机占用的时间百分比。

#### iostat

iostat命令被用于监视系统输入输出设备和CPU的使用情况。它的特点是汇报磁盘活动统计情况，同时也会汇报出CPU使用情况。同vmstat一样，iostat也有一个弱点，就是它不能对某个进程进行深入分析，仅对系统的整体情况进行分析。

```shell
## 命令格式:
iostat [ 选项 ] [ <时间间隔> [ <次数> ] ]

## 选项说明：
-c：仅显示CPU使用情况；
-d：仅显示设备利用率；
-k：显示状态以千字节每秒为单位，而不使用块每秒；
-m：显示状态以兆字节每秒为单位；
-p：仅显示块设备和所有被使用的其他分区的状态；
-t：显示每个报告产生时的时间；
-x：显示扩展状态。

## 参数说明：
间隔时间：每次报告的间隔时间（秒）；
次数：显示报告的次数。

## 常用命令
iostat -c 1 1

iostat -x /dev/sda1

iostat -d -m 2 2

iostat -d -k 1 10  ##参数 -d 表示，显示设备（磁盘）使用状态；-k某些使用block为单位的列强制使用Kilobytes为单位；1 10表示，数据显示每隔1秒刷新一次，共显示10次。

iostat -d -x -k 1 10 

iostat -c 1 10
```

输出数据说明：

1. Device: 监控设备名称
2. rrqm/s: 每秒需要读取需求的数量
3. wrqm/s: 每秒需要写入需求的数量
4. r/s：每秒实际读取需求的数量
5. w/s: 每秒实际写入需求的数量
6. rsec/s: 每秒读取区段的数量
7. wsec/s: 每秒写入区段的数量
8. rkB/s: 每秒实际读取的大小，单位为KB
9. wkB/s: 每秒实际写入的大小，单位为KB
10. avgrq-sz: 需求的平均大小区段
11. avgqu-sz: 需求的平均队列长度
12. await: 等待I/O平均时间(毫秒)
13. svctm: I/O需求完成的平均时间
14. util: 被I/O需求消耗的CPU百分比

#### sar

sar命令是Linux下系统运行状态统计工具，它将指定的操作系统状态计数器显示到标准输出设备。sar工具将对系统当前的状态进行取样，然后通过计算数据和比例来表达系统的当前运行状态。它的特点是可以连续对系统取样，获得大量的取样数据。取样数据和分析的结果都可以存入文件，使用它时消耗的系统资源很小。

```shell
## 命令格式:
sar [ 选项 ] [ <时间间隔> [ <次数> ] ]

## 选项说明：
[ -A ] [ -b ] [ -B ] [ -C ] [ -d ] [ -h ] [ -m ] [ -p ] [ -q ] [ -r ] [ -R ]
[ -S ] [ -t ] [ -u [ ALL ] ] [ -v ] [ -V ] [ -w ] [ -W ] [ -y ]
[ -I { [,…] | SUM | ALL | XALL } ] [ -P { [,…] | ALL } ]
[ -j { ID | LABEL | PATH | UUID | … } ] [ -n { [,…] | ALL } ]
[ -o [ ] | -f [ ] ] [ –legacy ]
[ -i ] [ -s [ ] ] [ -e [ ] ]

-A：显示所有的报告信息；
-b：显示I/O速率；
-B：显示换页状态；
-c：显示进程创建活动；
-d：显示每个块设备的状态；
-e：设置显示报告的结束时间；
-f：从指定文件提取报告；
-i：设状态信息刷新的间隔时间；
-P：报告每个CPU的状态；
-R：显示内存状态；
-u：显示CPU利用率；
-v：显示索引节点，文件和其他内核表的状态；
-w：显示交换分区状态；
-x：显示给定进程的状态。

## 主选项和报告：
-b I/O 和传输速率信息状况
-B 分页状况
-d 块设备状况
-I { <中断> | SUM | ALL | XALL } 中断信息状况
-m 电源管理信息状况
-n { <关键词> [,…] | ALL } 网络统计信息

## 关键词可以是：
DEV 网卡
EDEV 网卡 (错误)
NFS NFS 客户端
NFSD NFS 服务器
SOCK Sockets (套接字) (v4)
IP IP 流 (v4)
EIP IP 流 (v4) (错误)
ICMP ICMP 流 (v4)
EICMP ICMP 流 (v4) (错误)
TCP TCP 流 (v4)
ETCP TCP 流 (v4) (错误)
UDP UDP 流 (v4)
SOCK6 Sockets (套接字) (v6)
IP6 IP 流 (v6)
EIP6 IP 流 (v6) (错误)
ICMP6 ICMP 流 (v6)
EICMP6 ICMP 流 (v6) (错误)
UDP6 UDP 流 (v6)
-q 队列长度和平均负载
-r 内存利用率
-R 内存状况
-S 交换空间利用率
-u [ ALL ]
CPU 利用率
-v Kernel table 状况
-w 任务创建与系统转换统计信息
-W 交换信息
-y TTY 设备状况

## 参数说明：
间隔时间：每次报告的间隔时间（秒）；
次数：显示报告的次数。

##常用命令

##网络统计信息
sar -n DEV 1 5  ##命令中 1 5 表示每一秒钟取 1 次值，一共取 5 次。每个网卡这 5 次取值的平均数据

##CPU 利用率
sar -u 1 3 ##命令中 1 3 表示每一秒钟取 1 次值，一共取 3 次。

##索引节点，文件和其他内核表的状态
sar -v 1 3

##内存利用率
sar -r 1 3

## 内存分页状况
sar -B 1 3

##I/O 和传输速率信息状况
sar -b 1 3

## 队列长度和平均负载
sar -q 1 3

##系统交换信息
sar -W 1 3

##块设备状况
sar -d 1 3

##输出统计的数据信息
sar -o sarfile.log -u 1 3
sadf -d sarfile.log
sadf -d sarfile.log | sed 's/;/,/g' > sarfile.csv
```

#### perf

perf的原理是这样的：每隔一个固定的时间，就在CPU上（每个核上都有）产生一个中断，在中断上看看，当前是哪个pid，哪个函数，然后给对应的pid和函数加一个统计值，这样，我们就知道CPU有百分几的时间在某个pid，或者某个函数上了。

```shell
## 常用命令
perf stat的作用是执行一个命令并收集其运行过程中的各个数据，它可以提供一个程序运行情况的总体概览。
perf stat hostname

perf top的作用是实时地显示系统当前的性能统计信息。
perf top

perf record -a -g -p 187862
perf report -i perf.data
```

#### lsof

lsof 工具能够查看这个列表对系统监测以及排错

```shell
lsof [参数] [文件]

-a                   列出打开文件存在的进程
-c<进程名>            列出指定进程所打开的文件
-g                   列出 GID 号进程详情
-d<文件号>            列出占用该文件号的进程
+d<目录>              列出目录下被打开的文件
+D<目录>              递归列出目录下被打开的文件
-n<目录>              列出使用 NFS 的文件
-i<条件>              列出符合条件的进程(4、6、协议、:端口、@ip)
-p<进程号>            列出指定进程号所打开的文件
-u                   列出 UID 号进程详情
-h                   显示帮助信息
-v                   显示版本信息


##查看使用某个文件的相关进程
lsof /usr/bin/nginx

##递归列出目录下被打开的文件
lsof +D /opt/gogs/

##查看某个进程号打开的所有文件
lsof -p 9876

##查看某个进程名打开的所有文件
lsof -c gogs

##查看某个用户打开的所有文件
lsof -u upfor

##列出除了某个用户外的被打开的文件信息
lsof -u ^root,^upfor /opt/gogs/gogs

##列出多个进程号对应的文件信息
lsof -p 9183,26655

##列出所有的网络连接信息
lsof -i

##列出所有 tcp、udp网络连接信息
lsof -i tcp/udp

##列出谁在使用某个端口
lsof -i :80
lsof -i tcp:8080

##列出所有网络文件系统
lsof -N

##某个用户组所打开的文件信息
lsof -g 501
```

#### pidstat

pidstat是sysstat工具的一个命令，用于监控全部或指定进程的cpu、内存、线程、设备IO等系统资源的占用情况。pidstat首次运行时显示自系统启动开始的各项统计信息，之后运行pidstat将显示自上次运行该命令以后的统计信息。用户可以通过指定统计的次数和时间来获得所需的统计信息。

```shell
pidstat [ 选项 ] [ <时间间隔> ] [ <次数> ]

##常用参数
-u：默认的参数，显示各个进程的cpu使用统计
-r：显示各个进程的内存使用统计
-d：显示各个进程的IO使用情况
-p：指定进程号
-w：显示每个进程的上下文切换情况
-t：显示选择任务的线程的统计信息外的额外信息
-T { TASK | CHILD | ALL }
这个选项指定了pidstat监控的。TASK表示报告独立的task，CHILD关键字表示报告进程下所有线程统计信息。ALL表示报告独立的task和task下面的所有线程。
注意：task和子线程的全局的统计信息和pidstat选项无关。这些统计信息不会对应到当前的统计间隔，这些统计信息只有在子线程kill或者完成的时候才会被收集。
-V：版本号
-h：在一行上显示了所有活动，这样其他程序可以容易解析。
-I：在SMP环境，表示任务的CPU使用率/内核数量
-l：显示命令名和所有参数

##常用命令

##查看所有进程的 CPU 使用情况（ -u -p ALL）
pidstat -u -p ALL

##cpu使用情况统计(-u)
pidstat -u

##内存使用情况统计(-r)
pidstat -r

##显示各个进程的IO使用情况（-d）
pidstat -d

##显示每个进程的上下文切换情况（-w）
pidstat -w -p 2831

##显示选择任务的线程的统计信息外的额外信息 (-t)
pidstat -t -p 2831

pidstat -T

pidstat -wt -u 1 3
```

#### strace

查看系统调用和花费的时间

```shell
-f ：除了跟踪当前进程外，还跟踪其子进程。
-o file ：将输出信息写到文件file中，而不是显示到标准错误输出（stderr）。
-p pid ：绑定到一个由pid对应的正在运行的进程。此参数常用来调试后台进程。
-r 参数的作用是：在每行输出前加上相对时间戳，即每执行一条系统调用所耗费的时间

## 常用的命令
strace -T -r -c -p $pid
```

### 查看当前CPU频率和性能模式

```shell
##查看当前频率
cat /proc/cpuinfo |grep MHz|uniq

##修改为高性能模式
sudo cpupower frequency-set -g performance
```

### 查看CPU物理、逻辑核的命令

```shell
#查看逻辑CPU个数：
cat /proc/cpuinfo |grep "processor"|sort -u|wc -l
24

#查看物理CPU个数：
grep "physical id" /proc/cpuinfo|sort -u|wc -l          
2

grep "physical id" /proc/cpuinfo|sort -u          
physical id   : 0
physical id   : 1

#查看每个物理CPU内核个数：
grep "cpu cores" /proc/cpuinfo|uniq
cpu cores    : 6

#每个物理CPU上逻辑CPU个数：
grep "siblings" /proc/cpuinfo|uniq
siblings    : 12

#判断是否开启了超线程：
#如果多个逻辑CPU的"physical id"和"core id"均相同，说明开启了超线程或者换句话说
#逻辑CPU个数 > 物理CPU个数 * CPU内核数  开启了超线程
#逻辑CPU个数 = 物理CPU个数 * CPU内核数  没有开启超线程

#cat cpuinfo 
#!/bin/bash

physicalNumber=0
coreNumber=0
logicalNumber=0
HTNumber=0

logicalNumber=$(grep "processor" /proc/cpuinfo|sort -u|wc -l)
physicalNumber=$(grep "physical id" /proc/cpuinfo|sort -u|wc -l)
coreNumber=$(grep "cpu cores" /proc/cpuinfo|uniq|awk -F':' '{print $2}'|xargs)
HTNumber=$((logicalNumber / (physicalNumber * coreNumber)))

echo "****** CPU Information ******"
echo "Logical CPU Number : ${logicalNumber}"
echo "Physical CPU Number : ${physicalNumber}"
echo "CPU Core Number   : ${coreNumber}"
echo "HT Number      : ${HTNumber}"
echo "*****************************"



#执行结果：
./cpuinfo  

****** CPU Information ******
Logical CPU Number : 24
Physical CPU Number : 2
CPU Core Number   : 6
HT Number      : 2
*****************************
```
