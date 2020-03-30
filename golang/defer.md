## defer

`defer`是Golang语言提供的一种用于注册延迟调用的机制：让函数或者语句可以在当前函数执行完毕后(包括通过`return`正常结束或者`panic`导致的异常结束)执行。

`defer`语句通常用于一些成对操作的场景：打开连接/关闭连接；加锁/释放锁；打开文件/关闭文件等。

`defer`在一些需要回收资源的场景非常有用，可以很方便地在函数结束前做一些清理操作。在打开资源语句的下一行，直接一句defer就可以在函数返回前关闭资源，可谓相当优雅。

```go
f, _ := os.Open("defer.txt")defer f.Close()
```

注意：以上代码，忽略了err, 实际上应该先判断是否出错，如果出错了，直接return. 接着再判断 `f`是否为空，如果 `f`为空，就不能调用 `f.Close()`函数了，会直接panic的。

### 正确使用defer

defer的使用其实非常简单：

```go
f,err := os.Open(filename)if err != nil {    panic(err)}
if f != nil {    defer f.Close()}
```

在打开文件的语句附近，用defer语句关闭文件。这样，在函数结束之前，会自动执行defer后面的语句来关闭文件。

当然，defer会有小小地延迟，对时间要求特别特别特别高的程序，可以避免使用它，其他一般忽略它带来的延迟。

### defer遇到常见的2个问题

#### `defer`的调用时机和多个`defer`的调用顺序

##### 调用的时机

理解如下这条语句：

```go
return xxx
```

上面这条语句经过编译之后，变成了三条指令：

```go
1. 返回值 = xxx
2. 调用defer函数
3. 空的return
```

1,3步才是`Return` 语句真正的命令，第2步是`defer`定义的语句，这里可能会操作返回值。

下面我们来看两个例子，试着将`return`语句和`defer`语句拆解到正确的顺序。

第一个例子：

```go
func f() (r int) {     
    t := 5     
    defer func() {       
        t = t + 5     
    }()     
    return t
}
```

拆解后：

```go
func f() (r int) {     
    t := 5
    // 1. 赋值指令     
    r = t
    // 2. defer被插入到赋值与返回之间执行，这个例子中返回值r没被修改过     
    func() {                 
        t = t + 5     
    }
    // 3. 空的return指令     
    return
}
```

这里第二步没有操作返回值r, 因此，main函数中调用f()得到5.

##### 调用的顺序

```go
func main() {
	for i := 0; i < 5; i++ {
		defer fmt.Println(i)
	}
}

$ go run main.go
4
3
2
1
0
```

`defer` 传入的函数不是在退出代码块的作用域时执行的，它只会在当前函数和方法返回之前被调用，可以看到`defer`是后进先出的方式执行的。

#### `defer`预计算参数

`defer` 关键字使用传值的方式传递参数时会进行预计算，导致不符合预期的结果；

Go 语言中所有的函数调用都是传值的，`defer` 虽然是关键字，但是也继承了这个特性。假设我们想要计算 `main` 函数运行的时间，可能会写出以下的代码：

```go
func main() {
	startedAt := time.Now()
	defer fmt.Println(time.Since(startedAt))
	
	time.Sleep(time.Second)
}

$ go run main.go
0s
```

然而上述代码的运行结果并不符合我们的预期，这个现象背后的原因是什么呢？经过分析，我们会发现调用 `defer` 关键字会立刻对函数中引用的外部参数进行拷贝，所以 `time.Since(startedAt)` 的结果不是在 `main` 函数退出之前计算的，而是在 `defer` 关键字调用时计算的，最终导致上述代码输出 0s。

想要解决这个问题的方法非常简单，我们只需要向 `defer` 关键字传入匿名函数：

```go
func main() {
	startedAt := time.Now()
	defer func() { fmt.Println(time.Since(startedAt)) }()
	
	time.Sleep(time.Second)
}

$ go run main.go
1s
```

虽然调用 `defer` 关键字时也使用值传递，但是因为拷贝的是函数指针，所以 `time.Since(startedAt)` 会在 `main` 函数执行前被调用并打印出符合预期的结果。

#### 总结

`defer`语句并不会马上执行，而是会进入一个栈，函数`return`前，会按先进后出的顺序执行。也说是说最先被定义的`defer`语句最后执行。先进后出的原因是后面定义的函数可能会依赖前面的资源，自然要先执行；否则，如果前面先执行，那后面函数的依赖就没有了。

在`defer`函数定义时，对外部变量的引用是有两种方式的，分别是作为函数参数和作为闭包引用。**作为函数参数，则在defer定义时就把值传递给`defer`，并被`cache`起来；作为闭包引用的话，则会在`defer`函数真正调用时根据整个上下文确定当前的值。**

`defer`后面的语句在执行的时候，函数调用的参数会被保存起来，也就是复制了一份。真正执行的时候，实际上用到的是这个复制的变量，因此如果此变量是一个“值”，那么就和定义的时候是一致的。如果此变量是一个“引用”，那么就可能和定义的时候不一致。



### `defer`配合`recover`

Go被诟病比较多的就是它的`error`, 经常是各种`error`满天飞。编程的时候总是会返回一个`error`, 留给调用者处理。如果是那种致命的错误，比如程序执行初始化的时候出问题，直接`panic`掉，省得上线运行后出更大的问题。

但是有些时候，我们需要从异常中恢复。比如服务器程序遇到严重问题，产生了`panic`, 这时我们至少可以在程序崩溃前做一些“扫尾工作”，如关闭客户端的连接，防止客户端一直等待等等。

`panic`会停掉当前正在执行的程序，不只是当前协程。在这之前，它会有序地执行完当前协程`defer`列表里的语句，其它协程里挂的`defer`语句不作保证。因此，我们经常在`defer`里挂一个`recover`语句，防止程序直接挂掉，这起到了 `try...catch`的效果。

注意，**`recover()`函数只在`defer`的上下文中才有效（且只有通过在`defer`中用匿名函数调用才有效），直接调用的话，只会返回 `nil`.**

```go
func main() {
	defer fmt.Println("defer main")
	var user = os.Getenv("USER_")
	
	go func() {
		defer func() {
			fmt.Println("defer caller")
			if err := recover(); err != nil {
				fmt.Println("recover success. err: ", err)
			}
		}()

		func() {
			defer func() {
				fmt.Println("defer here")
			}()

			if user == "" {
				panic("should set user env.")
			}

			// 此处不会执行
			fmt.Println("after panic")
		}()
	}()

	time.Sleep(100)
	fmt.Println("end of main function")
}
```

上面的panic最终会被recover捕获到。这样的处理方式在一个http server的主流程常常会被用到。一次偶然的请求可能会触发某个bug, 这时用recover捕获panic, 稳住主流程，不影响其他请求。

## 引用

* [defer关键字的原理](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-defer/)
* [Golang之轻松化解defer的温柔陷阱](https://mp.weixin.qq.com/s/txj7jQNki_8zIArb9kSHeg)

