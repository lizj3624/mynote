---
title: "Linux的内存管理简介"
date: 2022-10-26T15:27:09+08:00
tags:
   - Linux内核
   - Linux内存管理
categories:
   - Linux内核
   - Linux内存管理
toc: true
---

## 前言

最近公司线上服务内存报警，经排查是Linux内存中`SReclaimable`导致内存报警，通过`slabtop`命令看一下，发现时`dentry`占比高，`dentry`(directory entry)就是目录项缓存，是内核用来高速查找文件的。

谷歌查询发现有类似的问题，主要是`curl`工具引用`NSS`的bug导致`dentry`泄漏，可以通过设置环境变量`NSS_SDB_USE_CACHE`或者升级高版本的`NSS`来解决，主要看[这篇文章](https://bugzilla.redhat.com/show_bug.cgi?id=1779325)。

由Linux内存引发的问题，因此系统学习一下Linux内存管理的知识非常有必要，以点带面系统学习。

## Linux内存管理

### 内存寻址

- 逻辑地址，每个逻辑地址都由一段(segment)和偏移量(offset)组成，偏移量指明了从段开始的地方到实际地址之间的距离。

- 线性地址，又称虚拟地址，是一个32个无符号整数，32位机器内存高达4GB，通常用十六进制数字表示，Linux进程的内存一般说的都是这个内存。

- 物理地址，用于内存芯片级内存单元寻址。它们与从CPU的地址引脚发送到内存总线上的电信号对应。

Linux中的内存控制单元(MMU)通过一种称为分段单元(segmentation unit)的硬件电路把一个逻辑地址转换成线性地址，接着，第二个称为分页单元(paging unit)的硬件电路把线性地址转换成一个物理地址。

![](./brief-linux-memory-01.png)

#### Linux分页机制

分页单元把线性地址转换成物理地址。

线性地址被分成以固定长度为单位的组，称为**页**(page)。页内部连续的线性地址被映射到连续的物理地址中。一般"页"既指一组线性地址，又指包含这组地址中的数据。

分页单元把所有的RAM分成固定长度的**页框**(page frame)，也成物理页。每一页框包含一个页(page)，也就是说一个页框的长度与一个页的长度一致。页框是主存的一部分，因此也是一个存储区域。区分一页和一个页框是很重要的，前者只是一个数据块，可以存放任何页框或者磁盘中。

把线性地址映射到物理地址的数据结构称为**页表**(page table)。页表存放在主存中，并在启用分页单元之前必须有内核对页表进行适当的初始化。

x86_64的Linux内核采用4级分页模型，一般一页4K，4种页表：

- 页全局目录

- 页上级目录

- 页中间目录

- 页表

页全局目录包含若干页上级目录，页上级目录又依次包含若干页中间目录的地址，而页中间目录又包含若干页表的地址。每个页表项指向一个页框。线性地址被分成5部分。

![](./linux-page.png)

Linux内核引用**TLB**(转换后援缓存器)用于加快线性地址的转换。

### 内存管理

#### NUMA

随着CPU进入多核时代，多核CPU通过一条数据总线访问内存延迟很大，因此**NUMA**架构应运而生，NUMA架构全称为非一致性内存架构 (Non Uniform Memory Architecture)，系统的物理内存被划分为几个节点(node)，每个node绑定不同的CPU核，本地CPU核访问本地内存node节点延迟最小。

可以通过`lscpu`查看NUMA与CPU核的关系。

#### 伙伴系统算法

内核应该为分配一组连续的页框而建立一种健壮、高效的分配策略，这个策略必须解决内存管理的外碎片问题。外碎片是指频繁地请求和释放不同大小的一组连续页框，必然导致在已分配的页框的块分散了许多小块的空闲页框。

内核采用著名的**伙伴系统算法**解决外碎片的问题。

内核试图把大小为b的一对空闲伙伴块合并为一个大小为2b的单独块。满足以下条件的两个块称为伙伴：

- 两个块具有相同的大小，计作b。

- 它们的物理地址是连续的。

- 第一块的第一页框的物理地址是2 x b x 2的次12方的倍数。

#### slab机制

**slab**机制的核心思想是以对象的观点来管理内存，主要是为了解决内部碎片。

slab分配器是基于对象进行管理的，所谓的对象就是内核中的数据结构（例如：task_struct,file_struct 等）。相同类型的对象归为一类，每当要申请这样一个对象时，slab分配器就从一个slab列表中分配一个这样大小的单元出去，而当要释放时，将其重新保存在该列表中，而不是直接返回给伙伴系统，从而避免内部碎片。slab分配器并不丢弃已经分配的对象，而是释放并把它们保存在内存中。slab分配对象时，会使用最近释放的对象的内存块，因此其驻留在cpu高速缓存中的概率会大大提高。

Linux中的高速缓存是用所谓 slab 层来实现的，slab层即内核中管理高速缓存的机制。

整个slab层的原理如下：

1. 可以在内存中建立各种对象的高速缓存(比如进程描述相关的结构 task_struct 的高速缓存)
2. 除了针对特定对象的高速缓存以外，也有通用对象的高速缓存
3. 每个高速缓存中包含多个 slab，slab用于管理缓存的对象
4. slab中包含多个缓存的对象，物理上由一页或多个连续的页组成

![](./slab.jpg)

> slab和伙伴系统(buddy)是上下级的调用关系，slab的内存来自buddy；它们都是内存分配器，只是buddy管理的是各ZONE映射区，slab管理的是buddy的各阶。

### 进程内存空间

所有进程都必须占用一定数量的内存，这些内存用来存放从磁盘载入的程序代码，或存放来自用户输入的数据等。内存可以提前静态分配和统一回收，也可以按需动态分配和回收。

对于普通进程对应的内存空间包含5种不同的数据区：

- 代码段(text)：程序代码在内存中的映射，存放函数体的二进制代码，通常用于存放程序执行代码(即CPU执行的机器指令)。
- 数据段(data)：存放程序中**已初始化且初值不为0**的**全局变量**和**静态局部变量**。数据段属于静态内存分配(静态存储区)，可读可写。
- BSS段(bss)：
  - 未初始化的全局变量和静态局部变量
  - 初始值为0的全局变量和静态局部变量(依赖于编译器实现)
  - 未定义且初值不为0的符号(该初值即common block的大小)
- 堆(heap)：动态分配的内存段，大小不固定，可动态扩张(malloc等函数分配内存)，或动态缩减(free等函数释放)；
- 栈(stack)：存放临时创建的局部变量；

![](./process-space.png)

![](./kernel_memory.jpg)

Linux内核是操作系统中优先级最高的，内核函数申请内存必须及时分配适当的内存，用户态进程申请内存被认为是不紧迫的，内核尽量推迟给用户态的进程动态分配内存。

- 请求调页，推迟到进程要访问的页不在RAM中时为止，引发一个缺页异常。

- 写时复制(COW)，父、子进程共享页框而不是复制页框，但是共享页框不能被修改，只有当父/子进程试图改写共享页框时，内核才将共享页框复制一个新的页框并标记为可写。

## Linux内存管理工具

### free

`free`命令可以监控系统内存

```shell
$ free -h
              total        used        free      shared  buff/cache   available
Mem:           31Gi        13Gi       8.0Gi       747Mi        10Gi        16Gi
Swap:         2.0Gi       321Mi       1.7Gi
```

`free` 指的是完全没有被用到的内存，而 Linux 认为内存不用也是浪费，因此会尽量“多”地把内存用来做各种缓存，提高系统的性能。在内存不够用时，它会释放缓存腾出空间给应用程序。因此早期没有 `available` 这项指标时，一般会认为 `free + buff/cache` 是系统当前的可用内存。在比较新的内核里，**会有 `available` 一项，它才是“可用内存”**

### top

`top`是Linux系统比较常用系统监控工具，涉及到内存，CPU，平均负载等指标。

![](./top.png)

- `VIRT` Virtual Memory Size (KiB)：进程使用的所有虚拟内存，包括代码（code）、数据（data）、共享库（shared libraries），以及被换出（swap out）到交换区和映射了（map）但尚未使用（未载入实体内存）的部分。
- `RES` Resident Memory Size (KiB)：进程所占用的所有实体内存（physical memory），不包括被换出到交换区的部分。
- `SHR` Shared Memory Size (KiB)：进程可读的全部共享内存，并非所有部分都包含在 `RES` 中。它反映了可能被其他进程共享的内存部分。

### smaps文件

`cat /proc/$pid/smaps`查看某进程虚拟内存空间的分布情况，比如heap占用了多少空间、文件映射（mmap）占用了多少空间、stack占用了多少空间？

```shell
0082f000-00852000 rw-p 0022f000 08:05 4326085    /usr/bin/nginx/sbin/nginx
Size:                140 kB
Rss:                 140 kB
Pss:                  78 kB
Shared_Clean:         56 kB
Shared_Dirty:         68 kB
Private_Clean:         4 kB
Private_Dirty:        12 kB
Referenced:          120 kB
Anonymous:            80 kB
AnonHugePages:         0 kB
Swap:                  0 kB
KernelPageSize:        4 kB
MMUPageSize:           4 kB
```

在**smaps**文件中，每一条记录（如下图所示）表示进程虚拟内存空间中一块连续的区域。其中第一行从左到右依次表示地址范围、权限标识、映射文件偏移、设备号、inode、文件路径。详细解释可以参见understanding-linux-proc-id-maps。

接下来8个字段的含义分别如下：

- Size：表示该映射区域在虚拟内存空间中的大小。
- Rss：表示该映射区域当前在物理内存中占用了多少空间　　　　　　
- Shared_Clean：和其他进程共享的未被改写的page的大小
- Shared_Dirty： 和其他进程共享的被改写的page的大小
- Private_Clean：未被改写的私有页面的大小。
- Private_Dirty： 已被改写的私有页面的大小。
- Swap：表示非mmap内存（也叫anonymous memory，比如malloc动态分配出来的内存）由于物理内存不足被swap到交换空间的大小。
- Pss：该虚拟内存区域平摊计算后使用的物理内存大小(有些内存会和其他进程共享，例如mmap进来的)。比如该区域所映射的物理内存部分同时也被另一个进程映射了，且该部分物理内存的大小为1000KB，那么该进程分摊其中一半的内存，即Pss=500KB。

### vmstat

`vmstat`是Virtual Meomory Statistics（虚拟内存统计）的缩写，可实时动态监视操作系统的虚拟内存、进程、CPU活动。

```shell
## 每秒统计3次
$ vmstat 1 3
procs -----------memory---------------- ---swap-- -----io---- --system-- -----cpu-----
 r  b    swpd   free   buff  cache       si   so    bi    bo   in   cs us sy id  wa st
 0  0      0 233483840 758304 20795596    0    0     0     1    0    0  0  0 100  0  0
 0  0      0 233483936 758304 20795596    0    0     0     0 1052 1569  0  0 100  0  0
 0  0      0 233483920 758304 20795596    0    0     0     0  966 1558  0  0 100  0  0
```

Procs（进程）:

- r: 运行队列中进程数量
- b: 等待IO的进程数量

Memory（内存）:

- swpd: 使用虚拟内存大小
- free: 可用内存大小
- buff: 用作缓冲的内存大小
- cache: 用作缓存的内存大小

Swap:

- si: 每秒从交换区写到内存的大小
- so: 每秒写入交换区的内存大小

IO：（现在的Linux版本块的大小为1024bytes）

- bi: 每秒读取的块数
- bo: 每秒写入的块数

system：

- in: 每秒中断数，包括时钟中断
- cs: 每秒上下文切换数

CPU（以百分比表示）

- us: 用户进程执行时间(user time)
- sy: 系统进程执行时间(system time)
- id: 空闲时间(包括IO等待时间)
- wa: 等待IO时间

### /proc/meminfo

Linux系统中`/proc/meminfo`这个文件用来记录了系统内存使用的详细情况。

```shell
$ cat /proc/meminfo
MemTotal:        8052444 kB
MemFree:         2754588 kB
MemAvailable:    3934252 kB
Buffers:          137128 kB
Cached:          1948128 kB
SwapCached:            0 kB
Active:          3650920 kB
Inactive:        1343420 kB
Active(anon):    2913304 kB
Inactive(anon):   727808 kB
Active(file):     737616 kB
Inactive(file):   615612 kB
Unevictable:         196 kB
Mlocked:             196 kB
SwapTotal:       8265724 kB
SwapFree:        8265724 kB
Dirty:               104 kB
Writeback:             0 kB
AnonPages:       2909332 kB
Mapped:           815524 kB
Shmem:            732032 kB
Slab:             153096 kB
SReclaimable:      99684 kB
SUnreclaim:        53412 kB
KernelStack:       14288 kB
PageTables:        62192 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:    12291944 kB
Committed_AS:   11398920 kB
VmallocTotal:   34359738367 kB
VmallocUsed:           0 kB
VmallocChunk:          0 kB
HardwareCorrupted:     0 kB
AnonHugePages:   1380352 kB
CmaTotal:              0 kB
CmaFree:               0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
DirectMap4k:      201472 kB
DirectMap2M:     5967872 kB
DirectMap1G:     3145728 kB
```

各字段说明：

- MemTotal: 所有内存(RAM)大小,减去一些预留空间和内核的大小。
- MemFree: 完全没有用到的物理内存，lowFree+highFree
- MemAvailable: 在不使用交换空间的情况下，启动一个新的应用最大可用内存的大小，计算方式：MemFree+Active(file)+Inactive(file)-(watermark+min(watermark,Active(file)+Inactive(file)/2))
- Buffers: 块设备所占用的缓存页，包括：直接读写块设备以及文件系统元数据(metadata)，比如superblock使用的缓存页。
- Cached: 表示普通文件数据所占用的缓存页。
- SwapCached: swap cache中包含的是被确定要swapping换页，但是尚未写入物理交换区的匿名内存页。那些匿名内存页，比如用户进程malloc申请的内存页是没有关联任何文件的，如果发生swapping换页，这类内存会被写入到交换区。
- Active: active包含active anon和active file
- Inactive: inactive包含inactive anon和inactive file
- Active(anon): anonymous pages（匿名页），用户进程的内存页分为两种：与文件关联的内存页(比如程序文件,数据文件对应的内存页)和与内存无关的内存页（比如进程的堆栈，用malloc申请的内存），前者称为file pages或mapped pages,后者称为匿名页。
- Inactive(anon): 见上
- Active(file): 见上
- Inactive(file): 见上
- SwapTotal: 可用的swap空间的总的大小(swap分区在物理内存不够的情况下，把硬盘空间的一部分释放出来，以供当前程序使用)
- SwapFree: 当前剩余的swap的大小
- Dirty: 需要写入磁盘的内存去的大小
- Writeback: 正在被写回的内存区的大小
- AnonPages: 未映射页的内存的大小
- Mapped: 设备和文件等映射的大小
- Slab: 内核数据结构slab的大小
- SReclaimable: 可回收的slab的大小
- SUnreclaim: 不可回收的slab的大小
- PageTables: 管理内存页页面的大小
- NFS_Unstable: 不稳定页表的大小
- VmallocTotal: Vmalloc内存区的大小
- VmallocUsed: 已用Vmalloc内存区的大小
- VmallocChunk: vmalloc区可用的连续最大快的大小

Linux 对内存的管理有多种视角：

系统内存 = 空闲内存 + 内核内存 + 用户内存

内核内存 = Slab + VmallocUsed + PageTables + KernelStack + HardwareCorrupted + Bounce + X

Slab = SUnreclaim + SReclaimable，其中 `SReclaimable` 指可回收部分

用户内存有两个视角：

- LRU 视角 = Active + Inactive + Unevictable + (HugePages_Total * Hugepagesize)
  - Active 与 Inactive 内存指的是活跃程度，如果内存紧张，会优先释放 Inactive 的内存
  - Active = Active(File) + Inactive(Anon)
  - Inactive = Inactive(File) + Inactive(Anon)
  - File-Backend 内存会与磁盘中的文件关联，于是如果内存不足时可以先写回磁盘释放内存；Anonymous 内存不与文件关联，因此除非有 swap 文件，否则无法释放
- 缓存视角 = Cached + AnonPages + Buffers + (HugePages_Total * Hugepagesize)

结合上述信息，可以看到可以释放的部分有：

- Slab 的 `SReclaimable`，是内核可释放的部分
- 所有的 File-Backend 内存 = Active(File) + Inactive(File)

通过`slabtop`命令查看那些对象占用slab内存高。

### 清除缓存

```shell
## 释放页缓存
echo 1 > /proc/sys/vm/drop_caches

## 释放目录和索引节点缓存（inode and dentry cache），清除Slab可回收缓存
echo 2 > /proc/sys/vm/drop_caches

## 同时释放 页、目录、索引节点缓存：
echo 3 > /proc/sys/vm/drop_caches
```

> 手动清除缓存可能会在一段时间内降低系统性能。原则上不推荐这么做，因为如果有需要，系统会自动释放出内存供其他程序使用。另外，手动清除Slab缓存是一个治标不治本的办法。因为问题不在Slab，而在于我们那个会引起Slab缓存飙涨的进程（如rsync）。实际操作的时候发现，清除缓存一段时间后，Slab缓存很快又会“反弹”回去。如果需要治本，要么搞定问题进程，要么修改系统配置。

-  vm.vfs_cache_pressure

系统在进行内存回收时，会先回收page cache, inode cache, dentry cache和swap cache。vfs_cache_pressure越大，每次回收时，inode cache和dentry cache所占比例越大。 vfs_cache_pressure默认是100，值越大inode cache和dentry cache的回收速度会越快，越小则回收越慢，为0的时候完全不回收(OOM!)。

- vm.min_free_kbytes

系统的”保留内存”的大小，”保留内存”用于低内存状态下的”atomic memory allocation requests”(eg. kmalloc + GFP_ATOMIC)，该参数也被用于计算开始内存回收的阀值，默认在开机的时候根据当前的内存计算所得，越大则表示系统会越早开始内存回收。 min_free_kbytes过大可能会导致OOM，太小可能会导致系统出现死锁等问题。

- vm.swappiness

该配置用于控制系统将内存swap out到交换空间的积极性，取值范围是[0, 100]。swappiness越大，系统的交换积极性越高，默认是60，如果为0则不会进行交换。

## 引用

1. [slab机制的原理](https://zhuanlan.zhihu.com/p/334607902)

2. [多核心Linux内核路径优化的不二法门之-slab与伙伴系统](https://blog.csdn.net/dog250/article/details/48487103)

3. [Linux内存管理](http://static.kancloud.cn/zhangyi8928/kernel/531037)

4. [万字整理，肝翻Linux内存管理所有知识点](https://zhuanlan.zhihu.com/p/519613267)

5. [Linux内存管理机制（最透彻的一篇）](https://zhuanlan.zhihu.com/p/474714333)

6. [Linux内存管理](http://gityuan.com/2015/10/30/kernel-memory/)

7. [linux虚拟内存与物理内存，内核态与用户态](https://blog.csdn.net/qq_35987777/article/details/106297198)

8. [/PROC/MEMINFO之谜](http://linuxperf.com/?p=142)

9. [socket 与 slab dentry](https://zhuanlan.zhihu.com/p/43133085)

10. [linux通过meminfo 与 slab 定位内存泄漏 - 简书](https://www.jianshu.com/p/a7af7c29c9e2)

11. [内存使用过高的调查过程](https://www.cnblogs.com/reve-wang/articles/7269652.html)

12. [Linux使用总结之：处理内存cache](https://blog.csdn.net/weixin_42488171/article/details/105587948)




