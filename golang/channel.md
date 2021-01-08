## 深入理解GO中channel

channel作为Go中核心的数据结构，是Go支持高并发goroutine之间通信的方式之一。

### channel基本知识

* 创建channel

```go
ch1 := make(chan int)  
ch2 := make(chan int, N) //带缓冲去的channel
```

* channel读写

```go
ch := make(chan int, 10)

// 读操作
x <- ch

// 写操作
ch <- x
```

* 关闭channel

```go
ch := make(chan int)

// 关闭
close(ch)
```

关于关闭 channel 有几点需要注意的是：

- 重复关闭 channel 会导致 panic。
- 向关闭的 channel 发送数据会 panic。
- 从关闭的 channel 读数据不会 panic，读出 channel 中已有的数据之后再读就是 channel 类似的默认值，比如 chan int 类型的 channel 关闭之后读取到的值为 0。



### channel常用方法

#### goroutine间通信

```go
func main() {
    x := make(chan int)
    go func() {
        x <- 1
    }()
    <-x
}
```

#### Select

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

`select`语句和`switch`语句一样，它不是循环，它只会选择一个case来处理，如果想一直处理channel，你可以在外面加一个无限的for循环：

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

#### range channel

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



### 引用

* [Go Channel 详解](https://colobu.com/2016/04/14/Golang-Channels/)
* [深入理解 Go Channel](http://legendtkl.com/2017/07/30/understanding-golang-channel/)
* [6.4 Channel](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/)
* [Go中的Channel](https://www.jianshu.com/p/15c94893124c)
* [Go中的Channel——range和select](https://www.jianshu.com/p/fe5dd2efed5d)
* [一文读懂channel设计](https://mp.weixin.qq.com/s/hVPIi4VVyRrO8T5Zd3E11A)
* [2020年度技术文章盘点之Channel篇](https://mp.weixin.qq.com/s/4HErPUVV9BAQahqQhoIefw)
