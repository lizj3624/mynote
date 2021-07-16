### Go中context

Go控制并发有两种经典的方式，一种是WaitGroup，另外一种就是Context。

#### 初识context

上面说的这种场景是存在的，比如一个网络请求Request，每个Request都需要开启一个goroutine做一些事情，这些goroutine又可能会开启其他的goroutine。所以我们需要一种可以跟踪goroutine的方案，才可以达到控制他们的目的，这就是Go语言为我们提供的Context，称之为上下文非常贴切，它就是goroutine的上下文。

如下是context控制多个goroutine的例子

```go
package main

import (
    "context"
    "fmt"
    "time"
)

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    go watch(ctx,"【监控1】")
    go watch(ctx,"【监控2】")
    go watch(ctx,"【监控3】")

    time.Sleep(10 * time.Second)
    fmt.Println("可以了，通知监控停止")
    cancel()
    //为了检测监控过是否停止，如果没有监控输出，就表示停止了
    time.Sleep(5 * time.Second)
}

func watch(ctx context.Context, name string) {
    for {
	select {
	case <-ctx.Done():
	    fmt.Println(name,"监控退出，停止了...")
	    return
	default:
	    fmt.Println(name,"goroutine监控中...")
	    time.Sleep(2 * time.Second)
	}
    }
}
```

示例中启动了3个监控goroutine进行不断的监控，每一个都使用了Context进行跟踪，当我们使用`cancel`函数通知取消时，这3个goroutine都会被结束。这就是Context的控制能力，它就像一个控制器一样，按下开关后，所有基于这个Context或者衍生的子Context都会收到通知，这时就可以进行清理操作了，最终释放goroutine，这就优雅的解决了goroutine启动后不可控的问题。

#### Context接口

```go
type Context interface {
    Deadline() (deadline time.Time, ok bool)

    Done() <-chan struct{}

    Err() error

    Value(key interface{}) interface{}
}
```

这个接口共有4个方法，了解这些方法的意思非常重要，这样我们才可以更好的使用他们。

`Deadline`方法是获取设置的截止时间的意思，第一个返回式是截止时间，到了这个时间点，Context会自动发起取消请求；第二个返回值ok==false时表示没有设置截止时间，如果需要取消的话，需要调用取消函数进行取消。

`Done`方法返回一个只读的chan，类型为`struct{}`，我们在goroutine中，如果该方法返回的chan可以读取，则意味着parent context已经发起了取消请求，我们通过`Done`方法收到这个信号后，就应该做清理操作，然后退出goroutine，释放资源。

`Err`方法返回取消的错误原因，因为什么Context被取消。

`Value`方法获取该Context上绑定的值，是一个键值对，所以要通过一个Key才可以获取对应的值，这个值一般是线程安全的。

以上四个方法中常用的就是`Done`了，如果Context取消的时候，我们就可以得到一个关闭的chan，关闭的chan是可以读取的，所以只要可以读取的时候，就意味着收到Context取消的信号了，以下是这个方法的经典用法。

```go
func Stream(ctx context.Context, out chan<- Value) error {
    for {
        v, err := DoSomething(ctx)
  	if err != nil {
  	    return err
  	}
  	select {
  	case <-ctx.Done():
  	    return ctx.Err()
  	case out <- v:
  	}
    }
}
```

#### Context的继承衍生

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc)
func WithValue(parent Context, key, val interface{}) Context
```

#### WithValue传递元数据

```go
var key string="name"

func main() {
    ctx, cancel := context.WithCancel(context.Background())
    //附加值
    valueCtx:=context.WithValue(ctx,key,"【监控1】")
    go watch(valueCtx)
    time.Sleep(10 * time.Second)
    fmt.Println("可以了，通知监控停止")
    cancel()
    //为了检测监控过是否停止，如果没有监控输出，就表示停止了
    time.Sleep(5 * time.Second)
}

func watch(ctx context.Context) {
    for {
        select {
	case <-ctx.Done():
	    //取出值
	    fmt.Println(ctx.Value(key),"监控退出，停止了...")
	    return
	default:
	    //取出值
	    fmt.Println(ctx.Value(key),"goroutine监控中...")
	    time.Sleep(2 * time.Second)
	}
    }
}
```

#### Context 使用原则

1. 不要把Context放在结构体中，要以参数的方式传递
2. 以Context作为参数的函数方法，应该把Context作为第一个参数，放在第一位。
3. 给一个函数方法传递Context的时候，不要传递nil，如果不知道传递什么，就使用context.TODO
4. Context的Value相关方法应该传递必须的数据，不要什么数据都使用这个传递
5. Context是线程安全的，可以放心的在多个goroutine中传递


### 引用

* [Go语言实战笔记（二十）| Go Context](https://www.flysnow.org/2017/05/12/go-in-action-go-context.html)
* [Golang 并发 与 context标准库](https://mp.weixin.qq.com/s/FJLH4o7Y1TG9I0seiNwR_w)
* [6.1 上下文 Context](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-context/)
* [Go Context 使用和源码分析](https://segmentfault.com/a/1190000019862527)
* [理解 golang 中的 context（上下文）包](https://studygolang.com/articles/13866?fr=sidebar)
* [go context](https://blog.golang.org/context)
* [go1.7中Context包的正确使用姿势 [译]](https://my.oschina.net/markz0928/blog/1833644)

