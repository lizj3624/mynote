golang 正常的 `struct` 就是普通的一个内存块，必定是占用一小块内存的，并且结构体的大小是要经过边界，长度的对齐的，但是“空结构体”是不占内存的，size 为 0；

> 提示：以下都是基于 go1.13.3 linux/amd64 分析。

普通的结构体定义如下：

```go
// 类型变量对齐到 8 字节；
type Tp struct {
    a uint16
    b uint32
}
```

按照内存对齐规则，这个结构体占用 8 个字节的内存。关于内存分配的基础知识可以翻看：[Golang 数据结构到底是怎么回事？gdb调一调？](http://mp.weixin.qq.com/s?__biz=MzU2MDcwNTg3OA==&mid=2247483768&idx=1&sn=5497331428066f19b2500fa6b59004ca&chksm=fc02b941cb7530574081972d30507b111a161c28ba16ed13b5beeab83744ce2088ad9627adf2&scene=21#wechat_redirect)，[golang 内存管理分析](http://mp.weixin.qq.com/s?__biz=MzU2MDcwNTg3OA==&mid=2247483858&idx=1&sn=ab90e35dbbc57ba851f89da14697b66a&chksm=fc02b9ebcb7530fdbdc01794a13f52c2166fceee481e1f3a433375f25e8a9850fa1de1d819e2&scene=21#wechat_redirect)。

空结构体：

```go
var s struct{}
// 变量 size 是 0 ；
fmt.Println(unsafe.Sizeof(s))
```

该空结构体的变量占用内存 0 字节。

本质上来讲，使用空结构体的初衷只有一个：节省内存，但是更多的情况，节省的内存其实很有限，这种情况使用空结构体的考量其实是：**根本不关心结构体变量的值**。

#### 特殊变量：zerobase 

空结构体是没有内存大小的结构体。这句话是没有错的，但是更准确的来说，其实是有一个特殊起点的，那就是 `zerobase` 变量，这是一个 `uintptr` 全局变量，占用 8 个字节。当在任何地方定义无数个 `struct {}` 类型的变量，编译器都只是把这个 `zerobase` 变量的地址给出去。换句话说，在 golang 里面，涉及到所有内存 size 为 0 的内存分配，那么就是用的同一个地址 `&zerobase` 。

举个例子：

```go
package main

import "fmt"

type emptyStruct struct {}

func main() {
 a := struct{}{}
 b := struct{}{}
 c := emptyStruct{}

 fmt.Printf("%p\n", &a)
 fmt.Printf("%p\n", &b)
 fmt.Printf("%p\n", &c)
}
```

dlv 调试分析一下：

```go
(dlv) p &a
(*struct {})(0x57bb60)
(dlv) p &b
(*struct {})(0x57bb60)
(dlv) p &c
(*main.emptyStruct)(0x57bb60)
(dlv) p &runtime.zerobase
(*uintptr)(0x57bb60)
```

小结：空结构体的变量的内存地址都是一样的。

#### 内存管理特殊处理

**mallocgc**

编译器在编译期间，识别到 `struct {}` 这种特殊类型的内存分配，会统统分配出 `runtime.zerobase` 的地址出去，这个代码逻辑是在 `mallocgc` 函数里面：

代码如下：

```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
    // 分配 size 为 0 的结构体，把全局变量 zerobase 的地址给出去即可；
 if size == 0 {
  return unsafe.Pointer(&zerobase)
 }
    // ... 
```

小结：golang 使用 `mallocgc` 分配内存的时候，如果 size 为 0 的时候，统一返回的都是全局变量 `zerobase` 的地址。

有这种全局唯一的特殊的地址也方便后面一些逻辑的特殊处理。

### 定义的各种姿势

#### 原生定义

```go
a := struct{}{}
```

`struct{}` 可以就认为是一种类型，a 变量就是 `struct {}` 类型的一种变量，地址为 `runtime.zerobase` ，大小为 0 ，不占内存。

#### 重定义类型

golang 使用 `type` 关键字定义新的类型，比如：

```go
type emptyStruct struct{}
```

定义出来的 `emptyStruct` 是新的类型，具有对应的 `type` 结构，但是性质 `struct{}` 完全一致，编译器对于 `emptryStruct` 类型的内存分配，也是直接给 `zerobase` 地址的。

#### 匿名嵌套类型

`struct{}` 作为一个匿名字段，内嵌其他结构体。这种情况是怎么样的？

**匿名嵌套方式一**

```go
type emptyStruct struct{}
type Object struct {
    emptyStruct
}
```

**匿名嵌套方式二**

```go
type Object1 struct {
    _ struct {}
}
```

记住一点，空结构体还是空结构体，类型变量本身绝对不分配内存（ size=0 ），所以编译器对以上的 `Object`，`Object1` 两种类型的处理和空结构体类型是一致的，分配地址为 `runtime.zerobase` 地址，变量大小为0，不占任何内存大小。

#### 内置字段

内置字段的场景没有什么特殊的，主要是地址和长度的对齐要考虑。还是只需要注意 3 个要点：

- 空结构体的类型不占内存大小；
- 地址偏移要和自身类型对齐；
- 整体类型长度要和最长的字段类型长度对齐；

我们分 3 种场景讨论这个问题：

**场景一：`struct {}` 在最前面**

这种场景非常好理解，`struct {}` 字段类型在最前面，这种类型不占空间，所以自然第二个字段的地址和整个变量的地址一致。

```go
// Object1 类型变量占用 1 个字节
type Object1 struct {
 s struct {}
 b byte
}

// Object2 类型变量占用 8 个字节
type Object2 struct {
 s struct {}
 n int64
}

o1 := Object1{ }
o2 := Object2{ }
```

内存怎么分配？

- `&o1` 和 `&o1.s` 是一致的，变量 `o1` 的内存大小对齐到 1 字节；
- `&o2` 和 `&o2.s` 是一致的，变量 `o2` 的内存大小对齐到 8 字节；

这种分配是满足对齐规则的，编译器也不会对这种 `struct {}` 字段做任何特殊的字节填充。

**场景二：`struct {}` 在中间**

```go
// Object1 类型变量占用 16 个字节
type Object1 struct {
 b  byte
 s  struct{}
 b1 int64
}

o1 := Object1{ }
```

- 按照对齐规则，变量 `o1` 占用 16 个字节；
- `&o1.s` 和 `&o1.b1` 相同；

编译器不会对 `struct { }` 做任何字节填充。

**场景三：`struct {}` 在最后**

这个场景稍微注意下，因为编译器遇到之后会做特殊的字节填充补齐，如下；

```go
type Object1 struct {
 b byte
 s struct{}
}

type Object2 struct {
 n int64
 s struct{}
}

type Object3 struct {
 n int16
 m int16
 s struct{}
}

type Object4 struct {
 n  int16
 m  int64
 s  struct{}
}

o1 := Object1 { }
o2 := Object2 { }
o3 := Object3 { }
o4 := Object4 { }
```

编译器在遇到这种 `struct {}` 在**最后一个字段**的场景，会进行特殊填充，`struct { }` 作为最后一个字段，会被填充对齐到前一个字段的大小，地址偏移对齐规则不变；

可以现在心里思考下，`o1`，`o2`，`o3`，`o4` 这四个对象的内存分配分别占多少空间？下面解密：

- 变量 `o1` 大小为 2 字节；
- 变量 `o2` 大小为 16 字节；
- 变量 `o3` 大小为 6 字节；
- 变量 `o4` 大小为 24 字节；

这种情况，需要先把 `struct {}` 按照前一个字段的长度分配 padding 内存，然后整个变量按照地址和长度的对齐规则不变。

### `struct {}` 作为 receiver

receiver 这个是 golang 里 struct 具有的基础特点。空结构体本质上作为结构体也是一样的，可以作为 receiver 来定义方法。

```go
type emptyStruct struct{}

func (e *emptyStruct) FuncB(n, m int) {
}
func (e emptyStruct) FuncA(n, m int) {
}

func main() {
 a := emptyStruct{}

 n := 1
 m := 2

 a.FuncA(n, m)
 a.FuncB(n, m)
}
```

receiver 这种写法是 golang 支撑面向对象的基础，本质上的实现也是非常简单，常规情况（普通的结构体）可以翻译成：

```go
func FuncA (e *emptyStruct, n, m int) {
}
func FuncB (e  emptyStruct, n, m int) {
}
```

**编译器只是把对象的值或地址作为第一个参数传给这个参数而已，就这么简单。** 但是在这里要提一点，空结构体稍微有一点点不一样，空结构体应该翻译成：

```go
func FuncA (e *emptyStruct, n, m int) {
}
func FuncB (n, m int) {
}
```

极其简单的代码，对应的汇编实际代码如下：

FuncA，FuncB 就这么简单，如下：

```go
00000000004525b0 <main.(*emptyStruct).FuncB>:
  4525b0: c3                    retq   

00000000004525c0 <main.emptyStruct.FuncA>:
  4525c0: c3                    retq    
```

main 函数

```go
00000000004525d0 <main.main>:
  4525d0: 64 48 8b 0c 25 f8 ff  mov    %fs:0xfffffffffffffff8,%rcx
  4525d9: 48 3b 61 10           cmp    0x10(%rcx),%rsp
  4525dd: 76 63                 jbe    452642 <main.main+0x72>
  4525df: 48 83 ec 30           sub    $0x30,%rsp
  4525e3: 48 89 6c 24 28        mov    %rbp,0x28(%rsp)
  4525e8: 48 8d 6c 24 28        lea    0x28(%rsp),%rbp
  4525ed: 48 c7 44 24 18 01 00  movq   $0x1,0x18(%rsp)
  4525f6: 48 c7 44 24 20 02 00  movq   $0x2,0x20(%rsp)
  4525ff: 48 8b 44 24 18        mov    0x18(%rsp),%rax
  452604: 48 89 04 24           mov    %rax,(%rsp)   // n 变量值压栈（第一个参数）
  452608: 48 c7 44 24 08 02 00  movq   $0x2,0x8(%rsp)  // m 变量值压栈（第二个参数）
  452611: e8 aa ff ff ff        callq  4525c0 <main.emptyStruct.FuncA>
  452616: 48 8d 44 24 18        lea    0x18(%rsp),%rax
  45261b: 48 89 04 24           mov    %rax,(%rsp)   // $rax 里面是 zerobase 的值，压栈（第一个参数）；
  45261f: 48 8b 44 24 18        mov    0x18(%rsp),%rax
  452624: 48 89 44 24 08        mov    %rax,0x8(%rsp)  // n 变量值压栈（第二个参数）
  452629: 48 8b 44 24 20        mov    0x20(%rsp),%rax
  45262e: 48 89 44 24 10        mov    %rax,0x10(%rsp)  // m 变量值压栈（第三个参数）
  452633: e8 78 ff ff ff        callq  4525b0 <main.(*emptyStruct).FuncB>
  452638: 48 8b 6c 24 28        mov    0x28(%rsp),%rbp
  45263d: 48 83 c4 30           add    $0x30,%rsp
  452641: c3                    retq   
  452642: e8 b9 7a ff ff        callq  44a100 <runtime.morestack_noctxt>
  452647: eb 87                 jmp    4525d0 <main.main>
```

通过这段代码证实几个点：

1. receiver 其实就是一种语法糖，本质上就是作为第一个参数传入函数；
2. receiver 为值的场景，不需要传空结构体做第一个参数，因为空结构体没有值；
3. receiver 为一个指针的场景，对象地址作为第一个参数传入函数，函数调用的时候，编译器传入 `zerobase` 的值（编译期间就可以确认）；

在二进制编译之后，一般 `e.FuncA` 的调用，第一个参数是直接压入 `&zerobase` 到栈里。

总结几个知识点：

- receiver 本质上是非常简单的一个通用思路，就是把对象值或地址作为第一参数传入函数；
- 函数参数压栈方式从前往后（可以调试看下）；
- 对象值作为 receiver 的时候，涉及到一次值拷贝；
- golang 对于值做 receiver 的函数定义，会根据现实需要情况可能会生成了两个函数，一个值版本，一个指针版本（思考：什么是“需要情况”？就是有 `interface` 的场景 ）；
- 空结构体在编译期间就能识别出来的场景，编译器会对既定的事实，可以做特殊的代码生成；

可以这么说，编译期间，关于空结构体的参数基本都能确定，那么代码生成的时候，就可以生成对应的静态代码。

程序 debug 技巧和工具介绍可以翻看：[golang 调试分析的高阶技巧](http://mp.weixin.qq.com/s?__biz=MzU2MDcwNTg3OA==&mid=2247484104&idx=1&sn=082bfb51db063d80aaa1ff2fb05bcae1&chksm=fc02baf1cb7533e767bc7c39dcc4178157dc36ad62efc90c0e084d2f13f86b571a00bbff0a8e&scene=21#wechat_redirect)。



空结构体 `struct{ }` 为什么会存在的核心理由就是为了**节省内存**。当你需要一个结构体，但是却丝毫不关系里面的内容，那么就可以考虑空结构体。golang 核心的几个复合结构 `map` ，`chan` ，`slice`都能结合 `struct{}` 使用。

### `map` & `struct{}`

`map` 和 `struct {}` 一般的结合姿势是这样的：

```go
// 创建 map
m := make(map[int]struct{})
// 赋值
m[1] = struct{}{}
// 判断 key 键存不存在
_, ok := m[1]
```

一般 `map` 和 `struct {}` 的结合使用场景是：只关心 key，不关注值。比如查询 key 是否存在就可以用这个数据结构，通过 `ok` 的值来判断这个键是否存在，`map` 的查询复杂度是 O(1) 的，查询很快。

你当然可以用 `map[int]bool` 这种类型来代替，功能也一样能实现，很多人考虑使用 `map[int]struct{}` 这种使用方式真的就是为了省点内存，当然大部分情况下，这种节省是不足道哉的，所以究竟要不要这样使用还是要看具体场景。

### `chan` & `struct{}`

`channel` 和 `struct{}` 结合是一个最经典的场景，`struct{}` 通常作为一个信号来传输，并不关注其中内容。chan 的分析在前几篇文章有详细说明。chan 本质的数据结构是一个管理结构加上一个 ringbuffer ，如果 `struct{}` 作为元素的话，ringbuffer 就是 0 分配的。

`chan` 和 `struct{}` 结合基本只有一种用法，就是**信号传递**，空结构体本身携带不了值，所以也只有这一种用法啦，一般来说，配合 no buffer 的 channel 使用。

```go
// 创建一个信号通道
waitc := make(chan struct{})

// ...
goroutine 1:
    // 发送信号: 投递元素
    waitc <- struct{}
    // 发送信号: 关闭
    close(waitc)

goroutine 2:
    select {
    // 收到信号，做出对应的动作
    case <-waitc:
    }    
```

这种场景我们思考下，是否一定是非 `struct{}` 不可？其实不是，而且也不多这几个字节的内存，所以这种情况真的就只是不关心 `chan` 的元素值而已，所以才用的 `struct{}`。

### `slice` & `struct{}`

形式上，`slice` 也结合 `struct{}` 。

```go
s := make([]struct{}, 100)
```

我们创建一个数组，无论分配多大，所占内存只有 24 字节（addr, len, cap），但实话说，这种用法没啥实用价值。

创建 slice 其实调用的是 `makeslice` 来分配内存，其中是调用 `malllocgc` ，而 `mallocgc` 我们知道在分配 size 为 0 的内存则是直接返回 `zerobase` 的地址而已。而 slice 在扩展的时候在遇到这种 size 为 0 的时候，也是直接返回 zerobase 的地址。

```go
func growslice(et *_type, old slice, cap int) slice {
    // 如果元素的 size 为 0，那么还是直接赋值了 zerobase 的地址；
    if et.size == 0 {
        return slice{unsafe.Pointer(&zerobase), old.len, cap}
    }
}
```

## 总结

1. 空结构体也是结构体，只是 size 为 0 的类型而已；
2. 所有的空结构体都有一个共同的地址：`zerobase` 的地址；
3. 空结构体可以作为 receiver ，receiver 是空结构体作为值的时候，编译器其实直接忽略了第一个参数的传递，编译器在编译期间就能确认生成对应的代码；
4. `map` 和 `struct{}` 结合使用常常用来节省一点点内存，使用的场景一般用来判断 key 存在于 `map`；
5. `chan` 和 `struct{}` 结合使用是一般用于信号同步的场景，用意并不是节省内存，而是我们真的并不关心 chan 元素的值；
6. `slice` 和 `struct{}` 结合好像真的没啥用。。。