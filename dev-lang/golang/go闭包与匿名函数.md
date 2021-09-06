- [闭包和匿名函数](#闭包和匿名函数)
  - [匿名函数的定义与实现](#匿名函数的定义与实现)
  - [闭包的定义](#闭包的定义)
    - [闭包的定义](#闭包的定义-1)
    - [引用环境的定义](#引用环境的定义)
  - [闭包的实现](#闭包的实现)
  - [关于闭包你需要掌握的几个点](#关于闭包你需要掌握的几个点)
    - [闭包与逃逸分析](#闭包与逃逸分析)
    - [闭包与外部函数的生命周期](#闭包与外部函数的生命周期)
    - [通过for循环的案例分析闭包对引用环境中变量的调用问题](#通过for循环的案例分析闭包对引用环境中变量的调用问题)
    - [当closure所在函数重新调用时，其closure是新的，其context引用的变量也是重新在heap定义过的](#当closure所在函数重新调用时其closure是新的其context引用的变量也是重新在heap定义过的)
  - [延迟调用与闭包](#延迟调用与闭包)
  - [总结](#总结)
# 闭包和匿名函数
最近在看go语言中的defer，里面涉及到了闭包，之前只是对闭包有了解，但是并没有深入的研究过，这次就深入研究一下吧。本文主要从如下几个点来展开描述，希望对你有所帮助~

> 1、匿名函数的定义与实现
> 2、闭包的定义 [穿插讲引用环境的定义]
> 3、闭包的实现
> 4、关于闭包你需要掌握的几个点：
>   （1）闭包与逃逸分析
>   （2）闭包与外部函数的生命周期
>   （3）通过for循环的案例分析闭包对引用环境中变量的调用问题
>   （4）当closure所在函数重新调用时，其closure是新的，其context引用的变量也是重新在heap定义过的。
> 5、延迟调用与闭包

## 匿名函数的定义与实现

   顾名思义，函数没有名字，跟正常的函数调用区别不大。 Go 里面匿名函数与正常的函数区别，参数的传递区别不大，只是在调用方面，匿名函数需要通过一个包装对象`func1.f`` 来调用匿名函数，这个过程通过 rbx 进行二次寻址来完成调用。理论上，匿名函数也会比正常函数性能要差。 具体可以参考[closure in go](https://links.jianshu.com/go?to=http%3A%2F%2Fsunisdown.me%2Fclosures-in-go.html)

## 闭包的定义

### 闭包的定义

闭包是由函数及其相关的引用环境组合而成的实体(即：闭包=函数+引用环境)。

### 引用环境的定义

> - 在函数式语言中，当内嵌函数体内引用到体外的变量时，将会把定义时涉及到的引用环境和函数体打包成一个整体（闭包）返回。现在给出引用环境的定义就容易理解了：引用环境是指在程序执行中的某个点所有处于活跃状态的约束（一个变量的名字和其所代表的对象之间的联系）所组成的集合。闭包的使用和正常的函数调用没有区别。

> - 由于闭包把函数和运行时的引用环境打包成为一个新的整体，所以就解决了函数编程中的嵌套所引发的问题。当每次调用包含闭包的函数时都将返回一个新的闭包实例，这些实例之间是隔离的，分别包含调用时不同的引用环境现场。不同于函数，闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。

## 闭包的实现

​闭包的实现也是一个匿名函数，一级指针在匿名函数里面指向“func1.f”， 在闭包中，指向闭包返回对象。闭包返回的包装对象是一个复合结构，里面包含匿名函数的地址，以及环境变量的地址。

## 关于闭包你需要掌握的几个点

### 闭包与逃逸分析

​闭包可能会导致变量逃逸到堆上来延长变量的生命周期，给 GC 带来压力。
![img](https:////upload-images.jianshu.io/upload_images/3763302-5034fe0243e13531.png?imageMogr2/auto-orient/strip|imageView2/2/w/644/format/webp)

testClosure.png

可以执行一下如下指令：
```go
go build -gcflags "-N -l -m" closure  
```

看下结果

![img](https:////upload-images.jianshu.io/upload_images/3763302-3d35c9f3b7538f04.png?imageMogr2/auto-orient/strip|imageView2/2/w/751/format/webp)

image.png

### 闭包与外部函数的生命周期

内函数对外函数的变量的修改，是对变量的引用。共享一个在堆上的变量。 变量被引用后，它所在的函数结束，这变量也不会马上被销毁。相当于变相延长了函数的生命周期。
看个例子：

```go
func AntherExFunc(n int) func() {
    n++
    return func() {
        fmt.Println(n)
    }
}

func ExFunc(n int) func() {
    return func() {
        n++
        fmt.Println(n)
    }
}

func main() {
    myAnotherFunc:=AntherExFunc(20)
    fmt.Println(myAnotherFunc)  //0x48e3d0  在这儿已经定义了n=20 ，然后执行++ 操作，所以是21 。
    myAnotherFunc()     //21 后面对闭包的调用，没有对n执行加一操作，所以一直是21
    myAnotherFunc()     //21

    myFunc:=ExFunc(10)
    fmt.Println(myFunc)  //0x48e340   这儿定义了n 为10
    myFunc()       //11  后面对闭包的调用，每次都对n进行加1操作。
    myFunc()       //12

}
```

### 通过for循环的案例分析闭包对引用环境中变量的调用问题

```go
func main() {                
    s := []string{"a", "b", "c"}                             
    for _, v := range s { 
        go func() {
            fmt.Println(v)
        }()                 
    }                        
    time.Sleep(time.Second * 1)                                                       
} 
```

结果会是什么呢？ a, b, c? 错了，结果是 c, c, c。为什么呢？
 这是因为for语句里面中闭包使用的v是外部的v变量，当执行完循环之后，v最终是c,所以输出了 c, c, c。 如果你去执行，有可能也不是这个结果。 输出这个结果的前提是“在主协程执行完for之后，定义的子协程 才开始执行，如果for过程中，子协程执行了，结果就可能不是c， c，c”。 输出的结果依赖于子协程执行时的那一刻，v是什么。

**之前 这个地方留了个疑问，是否是主协程中有闭包时，应该会先把闭包定义好，先不执行，当主协程执行完之后，再执行闭包所在的协程，此时协程使用环境变量v？？？**
 答案： 不是。
 看下面的程序便知：



```go
 func main() {        
      s := []string{"a", "b", "c"}
    for _, v := range s {
        go func() {
            fmt.Println(v)
        }()
        time.Sleep(time.Second * 3)
    }
    fmt.Println("main routine")
    time.Sleep(time.Second * 1)    // 阻塞模式
}
```

此时输出的就是 a, b, c , main routine
 为什么这次有正常了呢？  这是因为在for循环中执行了sleep， 让每次for循环中新定义的子协程有时间执行，子协程执行时获取环境中的变量v, 那么每次就会是本次循环执行时变量v的实际值。

如果想输出b, b, c 呢？可以看下面的代码：

```go
func main() {
  s := []string{"a", "b", "c"}
    for k, v := range s {
        go func() {
            fmt.Println(v)
        }()
        // 在执行到第二次时，sleep一会，此时已经定义的两个子协程得到执行，而此时的v为b
        if k == 1 {
            time.Sleep(time.Second * 3)
        }
    }
    fmt.Println("main routine")
    time.Sleep(time.Second * 1)    // 阻塞模式
}
```

最开始的for程序如果想输出a,b,c还有一种解决方法：
 只需要每次将变量v的拷贝传进函数即可，但此时就不是使用的上下文环境中的变量了。

```go
func main() {                
    s := []string{"a", "b", "c"}                             
    for _, v := range s { 
        go func(v string) {
            fmt.Println(v)
        }(v)   //每次将变量 v 的拷贝传进函数                 
    }                        
    select {}                                                      
}  
```

### 当closure所在函数重新调用时，其closure是新的，其context引用的变量也是重新在heap定义过的

```css
func main() {
    nextInt := intSeq()
    println(unsafe.Pointer(&nextInt))
    println(nextInt()) // 1
    println(nextInt()) // 2
    println(nextInt()) // 3

    newInts := intSeq()
    println(unsafe.Pointer(&newInts))
    println(newInts()) // 1

}

func intSeq() func() int {
    i := 0
    println("closure init--------------")
    println(unsafe.Pointer(&i))
    return func() int {
        i += 1
        println("in closure")
        println(unsafe.Pointer(&i))
        return i
    }
}
```

看下结果：



```kotlin
closure init--------------
0xc00001a068
0xc000042778
in closure
0xc00001a068
1
in closure
0xc00001a068
2
in closure
0xc00001a068
3
closure init--------------
0xc00001a080
0xc000042780
in closure
0xc00001a080
1
```

## 延迟调用与闭包

defer 调用会在当前函数执行结束前才被执行，这些调用被称为延迟调用 。defer 中使用匿名函数依然是一个闭包。

```go
package main

import "fmt"

func main() {
    x, y := 1, 2

    defer func(a int) { 
        fmt.Printf("x:%d,y:%d\n", a, y)  // y 为闭包引用
    }(x)      // 复制 x 的值

    x += 100
    y += 100
    fmt.Println(x, y)
}
```

输出结果是



```css
101 102
x:1,y:102
```

最开始这个地方我有个疑问： 为什么在defer中的x是1而不是101呢？其实是原因是 在defer定义时 已经将x的拷贝 1 复制给了defer, defer执行时使用的是当时defer定义时x的拷贝，而不是当前环境中x的值。

## 总结

本文分析了匿名函数、闭包的定义与实现，然后通过实例分析了闭包的一些重要指示以及易错点，希望对你有用~