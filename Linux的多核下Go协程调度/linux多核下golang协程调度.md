### Linux多核下golang协程调度

在将Linux多核下golang协程调度前，先将几个概念： 多核CPU，进程、线程、并发、并行、taskset、golang协程

#### 多核CPU

多核CPU是指将多个CPU集成到单个集成电路芯片中，因此主板的单个socket也可以适应这样的CPU，不需要去更改一些硬件结构。一个双核CPU有两个中央处理单元，操作系统看到是两个核心，不同的进程/线程可以在不同的核同时执行，加快系统的处理速度。两个核心都在同一芯片上，它们之间的通信也很快，系统也会有更小的延时。

![多核CPU]([https://github.com/lizj3624/mydoc/blob/master/Linux%E7%9A%84%E5%A4%9A%E6%A0%B8%E4%B8%8BGo%E5%8D%8F%E7%A8%8B%E8%B0%83%E5%BA%A6/pictures/%E5%A4%9A%E6%A0%B8CPU.png](https://github.com/lizj3624/mydoc/blob/master/Linux的多核下Go协程调度/pictures/多核CPU.png))

**超线程（hyper-threading）**

**超线程(hyper-threading)其实就是同时多线程(simultaneous multi-theading),是一项允许一个CPU执行多个控制流的技术。**它的原理很简单，就是把一颗CPU当成两颗来用，将一颗具有超线程功能的物理CPU变成两颗逻辑CPU，而逻辑CPU对操作系统来说，跟物理CPU并没有什么区别。因此，操作系统会把工作线程分派给这两颗（逻辑）CPU上去执行，让（多个或单个）应用程序的多个线程，能够同时在同一颗CPU上被执行。注意：两颗逻辑CPU共享单颗物理CPU的所有执行资源。因此，我们可以认为，**超线程技术就是对CPU的虚拟化**。Hyper-threading 使操作系统认为处理器的核心数是实际核心数的2倍，因此如果有4个核心的处理器，操作系统会认为处理器有8个核心。。超线程需要CPU、主板芯片、BIOS、OS的支持。



[多个CPU和多核CPU以及超线程（Hyper-Threading）](https://www.cnblogs.com/jokerjason/p/8926905.html)

[多核 CPU 和多个 CPU 有何区别-知乎](https://www.zhihu.com/question/20998226)

[LINUX内核调度算法（3）–多核系统的负载均衡]([http://www.taohui.pub/2015/01/27/linux%e5%86%85%e6%a0%b8%e8%b0%83%e5%ba%a6%e7%ae%97%e6%b3%95%ef%bc%883%ef%bc%89-%e5%a4%9a%e6%a0%b8%e7%b3%bb%e7%bb%9f%e7%9a%84%e8%b4%9f%e8%bd%bd%e5%9d%87%e8%a1%a1/](http://www.taohui.pub/2015/01/27/linux内核调度算法（3）-多核系统的负载均衡/))

#### 进程

程序是存放在磁盘上的一个可执行的二进制文件，程序的执行实例就是进程`process`，可以理解为进程就是程序的一个具体实现，`Linux`下每个进程都是唯一一个进程号`PID`，来表示一个进程。进程是资源分配的基本单位，资源主要是CPU，内存，文件等。一个进程至少需要一个线程作为它的指令执行体，一个进程中可以有多个线程。由于进程比较重量，占据独立的内存，所以上下文进程间的切换开销（栈、寄存器、虚拟内存、文件句柄等）比较大，但相对比较稳定安全。

#### 线程

线程是轻量级的进程，一个进程中可以包含多个线程，是CPU调度和分派的基本单位，线程可以分为内核级线程和用户级线程，线程包含了表示在进程内执行环境必须的信息，其中线程ID、一组寄存器值、栈、调度优先级和策略、信号、errno变量以及线程私有数据，进程的所有信息对其内的线程都是共享的，包含可执行的程序文本、程序全局内存、堆内存、栈以及文件表述符。每个线程都有一个线程ID。线程间通信主要通过共享内存，上下文切换很快，资源开销较少，但相比进程不够稳定容易丢失数据。

#### 并发和并行

在单核CPU上，Linux调度器通过分时机制调度进程/线程在CPU上执行，但是给使用者来看程序好像是同时执行的，这个叫做并发，而实际上CPU在同一个时刻只能有一个进程/线程执行。在多核CPU上，多个进程/线程可以在多个CPU核执行，这个叫做并行。

#### taskset

在Linux下，可以通过`taskset`命令将执行的进程绑定到一个或者多个CPU核，实现CPU亲和性

```shell
taskset -cp 1,2,3 2345   #将pid为2345的进程绑定在1，2，3核上
taskset -c 1-10 ./bin/mytest  #./bin/mytest运行时绑定在1-10个核上
```

#### Go协程

go的协程`Goroutine`是基于底层操作系统的线程实现的，比系统线程更轻量级，占用资源更少，通过`Go Scheduler`调度，go协程可以在多核CPU上并行运行，实现高并发。

调度器的三个抽象概念：G、M、P

- **G：**代表一个 goroutine，每个 goroutine 都有自己独立的栈存放当前的运行内存及状态。可以把一个G当做一个任务。
- **M:** 代表内核线程(Pthread)，它本身就与一个内核线程进行绑定，goroutine 运行在M上。
- **P：**代表一个处理器，可以认为一个“有运行任务”的P占了一个CPU线程的资源，且只要处于调度的时候就有P。

> 注：内核线程和 CPU 线程的区别，在系统里可以有上万个内核线程，但 CPU 线程并没有那么多，CPU 线程也就是 Top 命令里看到的 CPU0、CPU1、CPU2......的数量。

go程序启动时，通过`GOMAXPROCS`设置CPU核数`P`的个数，内核线程`M`绑定在`P`上，`P`中有一个协程`G`的队列，`M`就从绑定的`P`的队列中获取`G`执行。

![go的P-M-G模型]([https://github.com/lizj3624/mydoc/blob/master/Linux%E7%9A%84%E5%A4%9A%E6%A0%B8%E4%B8%8BGo%E5%8D%8F%E7%A8%8B%E8%B0%83%E5%BA%A6/pictures/P-M-G%E6%A8%A1%E5%9E%8B.jpg](https://github.com/lizj3624/mydoc/blob/master/Linux的多核下Go协程调度/pictures/P-M-G模型.jpg))

Go调度详解资料

* [详尽干货！从源码角度看 Golang 的调度](https://mp.weixin.qq.com/s/laxAshXPQvzRhFg3RtZJOw)
* [Goroutine调度](]https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-goroutine/)
* [Goroutine 并发调度模型深度解析之手撸一个高性能 goroutine 池](https://taohuawu.club/high-performance-implementation-of-goroutine-pool)

taskset绑定Go程序

Go程序在1.5版本以后，默认设置系统所有的CPU核数`runtime.GOMAXPROCS(runtime.NumCPU())`，可以通过``runtime.GOMAXPROCS`，可以通过`taskset`将Go程序绑定在某几个核上，实现CPU亲和性，同时可以跟其他程序将CPU隔离开，不会相互抢占CPU资源，尽量将`taskset`绑定CPU个数与`runtime.GOMAXPROCS`设置个数一致。

taskset绑定

```shell
taskset -c 1-10 ./mygopro
```

通过`GOMAXPROCS`设置CPU核数

```shell
func init() {
    runtime.GOMAXPROCS(10)
}
```



