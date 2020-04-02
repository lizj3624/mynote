## 深入理解Go Interface

Golang中提高的Interface（接口）类型可以实现类似面向对象的多态的特性，和Java的面向对象编程不同的是Golang不强制"面向对象"，它使用一种[鸭子类型](https://zh.wikipedia.org/wiki/%E9%B8%AD%E5%AD%90%E7%B1%BB%E5%9E%8B)的方式让动态类型称为可能。

### 鸭子类型

在 Go 中没有 `implements` 和 `extends` 这种关键字，这对我们而言反倒轻松了一些，它认为 Go 的接口就像鸭子测试里的描述：

> 当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。

在鸭子类型中，**关注点在于对象的行为**，能作什么；而不是关注对象所属的类型。例如，在不使用鸭子类型的语言中，我们可以编写一个函数，它接受一个类型为 “鸭子” 的对象，并调用它的 “走” 和 “叫” 方法。

在使用鸭子类型的语言中，这样的一个函数可以接受一个任意类型的对象，并调用它的 “走” 和 “叫” 方法。如果这些需要被调用的方法不存在，那么将引发一个运行时错误。

任何拥有这样的正确的 “走” 和 “叫” 方法的对象都可被函数接受的这种行为引出了以上表述，这种决定类型的方式因此得名。

### Go的interface

在`Go` 中，`interface` 是一组`method`的集合，是`duck-type programming`的一种体现。不关心属性（数据），只关心行为（方法）。具体使用中你可以自定义自己的`struct`，并提供特定的`interface`里面的`method`就可以把它当成`interface`来使用。下面是一种`interface`的典型用法，定义函数的时候参数定义成`interface`，调用函数的时候就可以做到非常的灵活。

举个例子

```go
package main

import "fmt"

type Duck interface {
    Swim()     // 游泳
    Feathers() // 羽毛
}

type MyDuck struct{}

func main() {
    var d Duck = &MyDuck{}
    d.Swim()
    d.Feathers()
}

func (p *MyDuck) Swim() {
    fmt.Printf("Swim\n")
}

func (p *MyDuck) Feathers() {
    fmt.Printf("Feathers\n")
}
```

定义接口`Duck`和结构体类型`Myduck`，`Myduck`实现了接口`Duck`的两个方法`Swim`和`Feathers`，就认为`MyDuck`实现了接口`Duck`，相比`Java`需要某个类显式的制定实现（`implements`）了某个接口，`Go`的接口隐式实现接口，可以理解`Go`的接口是**非侵入式的**。

### Go接口的特点

#### 泛型编程

Go中没有是没有泛型编程的，但是可以通过interface实现泛型编程，就是用前面说的鸭子类型实现，Go中的`Sort`函数就是通过接口实现的泛型编程，我们看一下代码

```go
package sort

// A type, typically a collection, that satisfies sort.Interface can be
// sorted by the routines in this package.  The methods require that the
// elements of the collection be enumerated by an integer index.
type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}

...

// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {
    // Switch to heapsort if depth of 2*ceil(lg(n+1)) is reached.
    n := data.Len()
    maxDepth := 0
    for i := n; i > 0; i >>= 1 {
        maxDepth++
    }
    maxDepth *= 2
    quickSort(data, 0, n, maxDepth)
}
```

只要实现`Len`、`Less`、`Swap`方法的都可以通过`Sort`函数进行排序，从而实现泛型编程。

#### 多态

多态是面向对象编程的特征之一，同一个消息可以根据发送对象的不同而采用多种不同的行为方式，只有在运行时才能判断引用对象的实际类型，根据实际类型的调用其相应的方法，在`c++/java`叫做**动态绑定**，在`Go`中叫做**动态派发**。

我们在改进`Duck`的代码，了解一下`Go`中的接口实现多态

```go
package main

import "fmt"

type Duck interface {
    Swim()     // 游泳
    Feathers() // 羽毛
}

type MyDuck struct{}
type YourDuck struct{}

func main() {
    var d Duck = &MyDuck{}
    d.Swim()
    d.Feathers()

    d = &YourDuck{}
    d.Swim()
    d.Feathers()
}

func (p *MyDuck) Swim() {
    fmt.Printf("MyDuck Swim\n")
}

func (p *MyDuck) Feathers() {
    fmt.Printf("MyDuck Feathers\n")
}

func (p *YourDuck) Swim() {
    fmt.Printf("YourDuck Swim\n")
}

func (p *YourDuck) Feathers() {
    fmt.Printf("YourDuck Feathers\n")
}
```

实现结果：

```go
MyDuck Swim
MyDuck Feathers
YourDuck Swim
YourDuck Feathers
```

#### 隐藏方法的具体实现

隐藏具体实现，这个很好理解。比如我设计一个函数给你返回一个 interface，那么你只能通过 interface 里面的方法来做一些操作，但是内部的具体实现是完全不知道的。Francesc 举了个 context 的例子。 context 最先由 google 提供，现在已经纳入了标准库，而且在原有 context 的基础上增加了：cancelCtx，timerCtx，valueCtx。语言的表达有时候略显苍白无力，看一下 context 包的代码吧。

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    c := newCancelCtx(parent)
    propagateCancel(parent, &c)
    return &c, func() { c.cancel(true, Canceled) }
}
```

表明上 WithCancel 函数返回的还是一个 Context interface，但是这个 interface 的具体实现是 cancelCtx struct。

```go
// newCancelCtx returns an initialized cancelCtx.
func newCancelCtx(parent Context) cancelCtx {
    return cancelCtx{
        Context: parent,
        done:    make(chan struct{}),
    }
}

// A cancelCtx can be canceled. When canceled, it also cancels any children
// that implement canceler.
type cancelCtx struct {
    Context     //注意一下这个地方

    done chan struct{} // closed by the first cancel call.
    mu       sync.Mutex
    children map[canceler]struct{} // set to nil by the first cancel call
    err      error                 // set to non-nil by the first cancel call
}

func (c *cancelCtx) Done() <-chan struct{} {
    return c.done
}

func (c *cancelCtx) Err() error {
    c.mu.Lock()
    defer c.mu.Unlock()
    return c.err
}

func (c *cancelCtx) String() string {
    return fmt.Sprintf("%v.WithCancel", c.Context)
}
```

尽管内部实现上下面三个函数返回的具体 struct （都实现了 Context interface）不同，但是对于使用者来说是完全无感知的。

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc)    //返回 cancelCtx
func WithDeadline(parent Context, deadline time.Time) (Context, CancelFunc) //返回 timerCtx
func WithValue(parent Context, key, val interface{}) Context    //返回 valueCtx
```

### 指针和结构体实现接口

```go
type Duck interface {
    Walk()
    Quack()
}

type Cat struct{}

//方法的接受者是指针
func (c *Cat) Walk() {
    fmt.Printf("catwalk\n")
}

func (c *Cat) Quack() {
    fmt.Printf("weow\n")
}

//方法的接受者是结构体
func (c Cat) Walk() {
    fmt.Printf("catwalk\n")
}

func (c Cat) Quack() {
    fmt.Printf("weow\n")
}
```

对 `Cat` 结构体来说，它在实现接口时可以选择接受者的类型，即结构体或者结构体指针，在初始化时也可以初始化成结构体或者指针。下面的代码总结了如何使用结构体、结构体指针实现接口，以及如何使用结构体、结构体指针初始化变量。

实现接口的类型和初始化返回的类型两个维度组成了四种情况，这四种情况并不都能通过编译器的检查：

|                      | 结构体实现接口 | 结构体指针实现接口 |
| -------------------- | -------------- | :----------------- |
| 结构体初始化变量     | 通过           | 不通过             |
| 结构体指针初始化变量 | 通过           | 通过               |

四种中只有『使用指针实现接口，使用结构体初始化变量』无法通过编译，其他的三种情况都可以正常执行。当实现接口的类型和初始化变量时返回的类型时相同时，代码通过编译是理所应当的：

- 方法接受者和初始化类型都是结构体；
- 方法接受者和初始化类型都是结构体指针；

而剩下的两种方式为什么一种能够通过编译，另一种无法通过编译呢？我们先来看一下能够通过编译的情况，也就是方法的接受者是结构体，而初始化的变量是结构体指针：

```go
type Cat struct{}

func (c Cat) Quack() {
	fmt.Println("meow")
}

func main() {
	var c Duck = &Cat{}
	c.Quack()
}
```

作为指针的 `&Cat{}` 变量能够**隐式地获取**到指向的结构体，所以能在结构体上调用 `Walk` 和 `Quack` 方法。我们可以将这里的调用理解成 C 语言中的 `d->Walk()` 和 `d->Speak()`，它们都会先获取指向的结构体再执行对应的方法。

但是如果我们将上述代码中方法的接受者和初始化的类型进行交换，代码就无法通过编译了：

```go
type Duck interface {
	Quack()
}

type Cat struct{}

func (c *Cat) Quack() {
	fmt.Println("meow")
}

func main() {
	var c Duck = Cat{}
	c.Quack()
}

$ go build interface.go
./interface.go:20:6: cannot use Cat literal (type Cat) as type Duck in assignment:
	Cat does not implement Duck (Quack method has pointer receiver)
```

编译器会提醒我们：`Cat` 类型没有实现 `Duck` 接口，`Quack` 方法的接受者是指针。这两个报错对于刚刚接触 Go 语言的开发者比较难以理解，如果我们想要搞清楚这个问题，首先要知道 Go 语言在[传递参数](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-function-call/)时都是传值的。

![golang-interface-method-receive](https://github.com/lizj3624/mynote/blob/master/golang/pictures/golang-interface-method-receiver.png)

**图实现接口的接受者类型**

如上图所示，无论上述代码中初始化的变量 `c` 是 `Cat{}` 还是 `&Cat{}`，使用 `c.Quack()` 调用方法时都会发生值拷贝：

- 如图左侧，对于 `&Cat{}` 来说，这意味着拷贝一个新的 `&Cat{}` 指针，这个指针与原来的指针指向一个相同并且唯一的结构体，所以编译器可以隐式的对变量解引用（dereference）获取指针指向的结构体；
- 如图右侧，对于 `Cat{}` 来说，这意味着 `Quack` 方法会接受一个全新的 `Cat{}`，因为方法的参数是 `*Cat`，编译器不会无中生有创建一个新的指针；即使编译器可以创建新指针，这个指针指向的也不是最初调用该方法的结构体；

上面的分析解释了指针类型的现象，当我们使用指针实现接口时，只有指针类型的变量才会实现该接口；当我们使用结构体实现接口时，指针类型和结构体类型都会实现该接口。当然这并不意味着我们应该一律使用结构体实现接口，这个问题在实际工程中也没那么重要，在这里我们只想解释现象背后的原因。

### 空接口

`Go`允许不带任何方法的`interface`，这种类型的`interface`叫`empty interface`(空接口)。所有类型都实现了 `empty interface`，因为任何一种类型至少实现了`0`个方法。

空接口是指没有定义任何接口方法的接口。**没有定义任何接口方法，意味着Go中的任意对象都可以实现空接口(因为没方法需要实现)，任意对象都可以保存到空接口实例变量中**。

空接口的定义方式：

```go
type empty_int interface {
}
```

通常会简写为`type empty_int interface{}`。

更常见的，会直接使用`interface{}`作为一种类型，表示空接口。例如：

```go
// 声明一个空接口实例
var i interface{}
```

再比如函数使用空接口类型参数：

```go
func myfunc(i interface{})
```

典型的应用场景是`fmt`包的`Println`方法，它能支持接收各种不同的类型的数据，并且输出到控制台,就是`interface{}`的功劳。下面我们看下案例：

```go
func Print(i interface{}) {
	fmt.Println(i)
}
func main() {
	var i interface{}
	i = "hello"
	Print(i)
	i = 100
	Print(i)
	i = 1.29
	Print(i)
}
```

但是`interface{}`类型的`slice`不能接受直接将`slice`赋值给接口，如下代码在编译时会有问题

```go
var dataSlice []int = foo()
var interfaceSlice []interface{} = dataSlice
// cannot use dataSlice (type []int) as type []interface { } in assignment
```

具体原因，官网 [wiki]([github.com/golang/go/w…](https://github.com/golang/go/wiki/InterfaceSlice)) 有描述,大致含义是，导致错误是有两个原因的：

- `[]interface{}` 并不是一个`interface`，它是一个`slice`,只是`slice`中的元素是interface
- `[]interface{}`类型的内存大小是在编译期间就确定的`(N*2)`,而其他切片类型的大小则为`N * sizeof(MyType)`,因此不发快速的将类型`[]MyType`分配给 `[]interface{}`。

### 判断interface类型

一个`interface`可被多种类型实现，有时候我们需要区分`interface`变量究竟存储哪种类型的值？类型断言提供对接口值的基础具体值的访问

```go
t := i.(T)
```

该语句断言接口值`i`保存的具体类型为`T`，并将T的基础值分配给变量`t`。如果i保存的值不是类型`T`，将会触发`panic`错误。为了避免`panic`错误发生，可以通过如下操作来进行断言检查

```go
t, ok := i.(T)
```

断言成功，`ok`的值为`true`,断言失败`t` 值为`T`类型的零值,并且不会发生`panic`错误。

```go
func main() {
	var i interface{}
	i = "hello"

	s := i.(string)
	fmt.Println(s)

	s, ok := i.(string)
	fmt.Println(s, ok)

	f, ok := i.(float64)
	fmt.Println(f, ok)

	i = 100
	t, ok := i.(int)
	fmt.Println(t, ok)

	t2 := i.(string) //panic
	fmt.Println(t2)
}
```

还有一种方便的方法来判断`interface`变量的具体类型，那就是利用`switch`语句。如下所示：

```go
func Print(i interface{}) {
	switch i.(type) {
	case string:
		fmt.Printf("type is string,value is:%v\n", i.(string))
	case float64:
		fmt.Printf("type is float32,value is:%v\n", i.(float64))
	case int:
		fmt.Printf("type is int,value is:%v\n", i.(int))
	}
}
func main() {
	var i interface{}
	i = "hello"
	Print(i)
	i = 100
	Print(i)
	i = 1.29
	Print(i)
}
```

灵活高效的 interface 动态类型,使 Go 语言在保持强静态类型的安全和高效的同时，也能灵活安全地在不同相容类型之间转换

### 引用

* [[浅入浅出Go 语言接口的原理](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-interface/)]
* [深入理解 Go Interface](http://legendtkl.com/2017/06/12/understanding-golang-interface/)

* [原来这才是 Go Interface](https://juejin.im/post/5d8877f1f265da03986c311c)
* [Go基础系列：空接口](https://studygolang.com/articles/16438)
* [golang里interface空指针](https://www.jianshu.com/p/97bfe8104e03?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com)

* [Golang interface接口深入理解](https://juejin.im/post/5a6873fd518825734501b3c5)

* [深度解密Go语言之关于 interface 的 10 个问题](https://mp.weixin.qq.com/s/EbxkBokYBajkCR-MazL0ZA)

