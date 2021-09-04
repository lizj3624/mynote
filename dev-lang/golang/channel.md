- [深入理解GO中channel](#深入理解go中channel)
  - [channel基本知识](#channel基本知识)
  - [channel常用方法](#channel常用方法)
    - [goroutine间通信](#goroutine间通信)
    - [Select](#select)
    - [range channel](#range-channel)
    - [timeout](#timeout)
    - [channel的内部实现](#channel的内部实现)
  - [引用](#引用)
# 深入理解GO中channel

channel作为Go中核心的数据结构，是Go支持高并发goroutine之间通信的方式之一。
## channel基本知识

1. 创建channel

```go
ch1 := make(chan int)  
ch2 := make(chan int, N) //带缓冲去的channel
```

2. channel读写

```go
ch := make(chan int, 10)

// 读操作
x <- ch

// 写操作
ch <- x
```

3. 关闭channel

```go
ch := make(chan int)

// 关闭channel, 一般是向channel写一个channel类型的零值, 设置channel=ni,
// 关闭channel不是必须的，不被引用channel会被GC回收的，close(ch)一般是用来通知其他协程某个任务已经完成了
close(ch)

//判断channel是否关闭
c, ok := <-ch    
//如果ok为false时，说明channel已经关闭
```

关于关闭 channel 有几点需要注意的是：

- 重复关闭 channel 会导致 panic。
- 向关闭的 channel 发送数据会 panic。
- 从关闭的 channel 读数据不会 panic，读出 channel 中已有的数据之后再读就是 channel 类似的默认值，比如 chan int 类型的 channel 关闭之后读取到的值为 0。

关闭 channel 时应该注意以下准则：

- 不要在读取端关闭 channel ，因为写入端无法知道 channel 是否已经关闭，往已关闭的 channel 写数据会 panic ；
- 有多个写入端时，不要再写入端关闭 channle ，因为其他写入端无法知道 channel 是否已经关闭，关闭已经关闭的 channel 会发生 panic ；
- 如果只有一个写入端，可以在这个写入端放心关闭 channel 。

## channel常用方法

### goroutine间通信

```go
func main() {
    x := make(chan int)
    go func() {
        x <- 1
    }()
    <-x
}
```

### Select
`select`语句选择一组可能的send操作和receive操作去处理。它类似`switch`,但是只是用来处理通讯(communication)操作。它的`case`可以是send语句，也可以是receive语句，亦或者`default`。`receive`语句可以将值赋值给一个或者两个变量。它必须是一个receive操作。

最多允许有一个`default case`,它可以放在case列表的任何位置，尽管我们大部分会将它放在最后。

```go
import "fmt"

func fibonacci(c, quit chan int) {
    x, y := 0, 1
    for {
        select {
	case c <- x:
	    x, y = y, x+y
	case <-quit:
	    fmt.Println("quit")
	    return
	}
    }
}

func main() {
    c := make(chan int)
    quit := make(chan int)
    go func() {
	for i := 0; i < 10; i++ {
	    fmt.Println(<-c)
	}
	quit <- 0
    }()
    fibonacci(c, quit)
}
```
如果有同时多个case去处理,比如同时有多个channel可以接收数据，那么Go会伪随机的选择一个case处理(pseudo-random)。如果没有case需要处理，则会选择`default`去处理，如果`default case`存在的情况下。如果没有`default case`，则`select`语句会阻塞，直到某个case需要处理。

需要注意的是，nil channel上的操作会一直被阻塞，如果没有default case,只有nil channel的select会一直被阻塞。

`select`语句和`switch`语句不一样，它不是循环，它只会选择一个case来处理，如果想一直处理channel，你可以在外面加一个无限的for循环：

```go
for {
    select {
    case c <- x:
	x, y = y, x+y
    case <-quit:
	fmt.Println("quit")
	return
    }
}
```

* 问题：select监听多个channel时，同时又多个channel有数据时，select只能随机出来其中一个，那么其他channel的数据会丢失吗？
不会丢失，下次循环时再获取channel中数据。

### range channel

range channel 可以直接取到 channel 中的值。当我们使用 range 来操作 channel 的时候，一旦 channel 关闭，channel 内部数据读完之后循环自动结束。

```go
func consumer(ch chan int) {
    for x := range ch {
        fmt.Println(x)
        ...
    }
}

func producer(ch chan int) {
  for _, v := range values {
      ch <- v
  }  
}
```

### timeout

`select`有很重要的一个应用就是超时处理。 因为上面我们提到，如果没有case需要处理，select语句就会一直阻塞着。这时候我们可能就需要一个超时操作，用来处理超时的情况。
下面这个例子我们会在2秒后往channel c1中发送一个数据，但是`select`设置为1秒超时,因此我们会打印出`timeout 1`,而不是`result 1`。

```go
import "time"
import "fmt"

func main() {
    c1 := make(chan string, 1)
    go func() {
        time.Sleep(time.Second * 2)
        c1 <- "result 1"
    }()

    select {
    case res := <-c1:
        fmt.Println(res)
    case <-time.After(time.Second * 1):
        fmt.Println("timeout 1")
    }
}


###timer.C
timer2 := time.NewTimer(time.Second)
go func() {
    <-timer2.C
    fmt.Println("Timer 2 expired")
}()
stop2 := timer2.Stop()
if stop2 {
    fmt.Println("Timer 2 stopped")
}
```

其实它利用的是`time.After`方法，它返回一个类型为`<-chan Time`的单向的channel，在指定的时间发送一个当前时间给返回的channel中。

### channel的内部实现

`Go`语言`channel`是`first-class`的，意味着它可以被存储到变量中，可以作为参数传递给函数，也可以作为函数的返回值返回。作为`Go`语言的核心特征之一，虽然`channel`看上去很高端，但是其实它仅仅就是一个数据结构而已，具体定义在 `$GOROOT/src/runtime/chan.go`里。如下：

```go
 1 type hchan struct {
 2   qcount uint   // 队列中的总数据
 3   dataqsiz uint   // 循环队列的大小
 4   buf unsafe.Pointer // 指向dataqsiz元素数组 指向环形队列
 5   elemsize uint16  // 
 6   closed uint32 
 7   elemtype *_type // 元素类型
 8   sendx uint // 发送索引
 9   recvx uint // 接收索引
10   recvq waitq // 接待员名单， 因recv而阻塞的等待队列。
11   sendq waitq // 发送服务员列表， 因send而阻塞的等待队列。
12   //锁定保护hchan中的所有字段，以及几个在此通道上阻止的sudogs中的字段。
13   //按住此锁定时不要更改另一个G的状态（尤其是不要准备G），因为这可能会导致死锁堆栈缩小。
14   lock mutex 
15 }
```

其核心是存放channel数据的环形队列，由qcount和elemsize分别指定了队列的容量和当前使用量。dataqsize是队列的大小。elemalg是元素操作的一个Alg结构体，记录下元素的操作，如copy函数，equal函数，hash函数等。如果是带缓冲区的chan，则缓冲区数据实际上是紧接着Hchan结构体中分配的。不带缓冲的 channel ，环形队列 size 则为 0。

```go
1 c = (Hchan*)runtime.mal(n + hint*elem->size);
```

另一重要部分是recvq和sendq两个双向链表，前者是等待读通道(<-channel)的goroutine队列，后者是等待写通道(channel <- xxx)的goroutine队列。若一个goroutine阻塞于channel了，那么它就被挂在recvq或sendq队列中。WaitQ是链表的定义，包含一个头结点和一个尾结点：

```go
1 struct    WaitQ
2 {
3     SudoG*    first;
4     SudoG*    last;
5 };
```

队列中的每个成员是一个SudoG结构体变量：

```go
1 struct    SudoG
2 {
3     G*    g;        // g和selgen构成
4     uint32    selgen;        // 指向g的弱指针
5     SudoG*    link;
6     int64    releasetime;
7     byte*    elem;        // 数据元素
8 };
```

SudoG里主要结构是一个g和一个elem。elem用于存储goroutine的数据。读通道时，数据会从Hchan的buf队列中拷贝到SudoG的elem域。写通道时，数据则是由SudoG的elem域拷贝到Hchan的队列中。

![img](https://img2018.cnblogs.com/blog/1069650/201911/1069650-20191119161656861-2110164001.png) 

- `buf`是有缓冲的channel所特有的结构，用来存储缓存数据。是个循环链表
- `sendx`和`recvx`用于记录`buf`这个循环链表中的发送或者接收的index
- `lock`是个互斥锁。

从最基本的开始-创建channel，创建一个缓冲channel

```go
1 ch := make(chan int, 3)  //
```

底层操作就是从Heap中分配一块内存，在内存中实例化一个`hchan`的结构体，并返回一个ch指针，使用 channel时，在函数之间的传递就是这个指针，这就是为什么函数传递中无需使用channel的指针，而直接用channel就行了，因为channel本身就是一个指针。　  　

基本的写channel操作，在底层运行时库中对应的是一个runtime.chansend函数。    

```go
chan <- value
//在运行时库中会执行：
void runtime·chansend(ChanType *t, Hchan *c, byte *ep, bool *pres, void *pc)
```

其中c就是`channel`，`ep`是取变量v的地址。这里的传值约定是调用者负责分配好`ep`的空间，仅需要简单的取变量地址就够了。`pres`参数是在`select`中的通道操作使用的。     

这个函数首先判断是同步或异步。同步`chan`不带缓冲区，可能写阻塞，而异步`chan`带缓冲区，只有缓冲区满才阻塞。在同步的情况下，首先查看`Hchan`结构体中的`recvq`链表时否为空，即是否有因为读该管道而阻塞的`goroutine`。如果有则可以正常写`channel`，否则操作会阻塞。           

`recvq`不为空时，将一个`SudoG`结构体出队列，将传给通道的数据(函数参数`ep`)拷贝到`SudoG`结构体中的`elem`域，并将`SudoG`中的`g`放到就绪队列中，状态置为`ready`，然后函数返回。如果`recvq`为空，将当前`goroutine`阻塞。此时将一个`SudoG`结构体挂到通道的`sendq`链表中，这个`SudoG`中的`elem`域是参数`eq`，`SudoG`中的`g`是当前的`goroutine`。当前`goroutine`会被设置为`waiting`状态并挂到等待队列中。    

　　异步时，如果缓冲区满，要将当前`goroutine`和数据一起作为`SudoG`结构体挂在`sendq`队列中。在`channel`缓冲区不满的情况，直接将数据放到`channel`的缓冲区中，调用者返回。     

实现细节：

- 当使用`send (ch <- xx)`或者`recv ( <-ch)`的时候，首先要**锁住**`hchan`这个结构体。（lock字段）；
- 向缓冲区写数据，按链表顺序存放在buf中，直到缓冲区满；
- 取数据的时候按链表顺序读取，符合FIFO的原则。

读写操作的细节都可以细化为：

- 第一，加锁
- 第二，把数据从goroutine中copy到“队列”中(或者从队列中copy到goroutine中）。
- 第三，释放锁

读`channel`操作也是类似的，对应的函数是`runtime.chansend`。基本过程类似。     

当协程尝试从未关闭的 `channel`中读取数据时，内部的操作如下：     

- 当 buf 非空时，此时 recvq 必为空，buf 弹出一个元素给读协程，读协程获得数据后继续执行，此时若 sendq 非空，则从 sendq 中弹出一个写协程转入 running 状态，待写数据入队列 buf ，此时读取操作 `<- ch` 未阻塞；
- 当 buf 为空但 sendq 非空时(不带缓冲的 channel)，则从 sendq 中弹出一个写协程转入 running 状态，待写数据直接传递给读协程，读协程继续执行，此时读取操作 `<- ch` 未阻塞；
- 当 buf 为空并且 sendq 也为空时，读协程入队列 recvq 并转入 blocking 状态，当后续有其他协程往 channel 写数据时，读协程才会重新转入 running 状态，此时读取操作 `<- ch` 阻塞。

类似的，当协程尝试往未关闭的`channel`中写入数据时，内部的操作如下：

- 当队列 recvq 非空时，此时队列 buf 必为空，从 recvq 弹出一个读协程接收待写数据，此读协程此时结束阻塞并转入 running 状态，写协程继续执行，此时写入操作 `ch <-` 未阻塞；
- 当队列 recvq 为空但 buf 未满时，此时 sendq 必为空，写协程的待写数据入 buf 然后继续执行，此时写入操作 `ch <-` 未阻塞；
- 当队列 recvq 为空并且 buf 为满时，此时写协程入队列 sendq 并转入 blokcing 状态，当后续有其他协程从 channel 中读数据时，写协程才会重新转入 running 状态，此时写入操作 `ch <-` 阻塞。

当关闭`non-nil channel`时，内部的操作如下：

- 当队列 recvq 非空时，此时 buf 必为空，recvq 中的所有协程都将收到对应类型的零值然后结束阻塞状态；
- 当队列 sendq 非空时，此时 buf 必为满，sendq 中的所有协程都会产生 panic ，在 buf 中数据仍然会保留直到被其他协程读取。

空通道是指将一个`channel`赋值为`nil`，或者定义后不调用`make`进行初始化。按照`Go`语言的语言规范，读写空通道是永远阻塞的。其实在函数`runtime.chansend`和`runtime.chanrecv`开头就有判断这类情况，如果发现参数c是空的，则直接将当前的`goroutine`放到等待队列，状态设置为`waiting`。

**读一个关闭的通道，永远不会阻塞，会返回一个通道数据类型的零值。这个实现也很简单，将零值复制到调用函数的参数ep中。写一个关闭的通道，则会panic。关闭一个空通道，也会导致panic。**

## 引用

* [Go Channel 详解](https://colobu.com/2016/04/14/Golang-Channels/)
* [深入理解 Go Channel](http://legendtkl.com/2017/07/30/understanding-golang-channel/)
* [6.4 Channel](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/)
* [Go中的Channel](https://www.jianshu.com/p/15c94893124c)
* [Go中的Channel——range和select](https://www.jianshu.com/p/fe5dd2efed5d)
* [一文读懂channel设计](https://mp.weixin.qq.com/s/hVPIi4VVyRrO8T5Zd3E11A)
* [2020年度技术文章盘点之Channel篇](https://mp.weixin.qq.com/s/4HErPUVV9BAQahqQhoIefw)
