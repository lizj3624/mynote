# golang切片详解

### 数组

Go的数组(Array)是长度相同，类型相同的一组数据组成，长度(len)和容量(cap)是相同的。

```go
arr := [4]int{2, 3, 4, 5}
arr := [...]int{4, 5, 6, 8}   #数组长度可以推导出
```

数组一旦定义后，就长度就固定了，在Go实际编程中，数组不如可以动态增长的切片(Slice)常用。

* 数组不需要显式的初始化
* 数组中数据是连续一段内存存储的

* 数组的零值是可以直接使用的
* 数组元素会自动初始化为其对应类型的零值

* 一个数组变量表示整个数组，它不是指向第一个元素的指针（不像 C 语言的数组）
* 当一个数组变量被赋值或者被传递的时候，实际上会复制整个数组，为了避免复制数组，你可以传递一个指向数组的指针，但是数组指针并不是数组。

![数组](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/pictures/array.png)

### 切片的定义

Go中的切片是长度可以动态增长，类型系统的一组数据组成。长度(len)和容量(cap)可以不一致，一般`len <= cap`。

```go
mySlice := []int{4, 5, 6, 7} #字面量定义

mySlice := make([]int, 10)  #这种是推荐的定义切片的方法
```



### 切片的本质

Go的切片的底层是具有数组的，编译期间的切片是 `Slice` 类型的，但是在运行时切片由如下的 `SliceHeader` 结构体表示，其中 `Data` 字段是指向数组的指针，`Len` 表示当前切片的长度，而 `Cap` 表示当前切片的容量，也就是 `Data` 数组的大小：

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```

`Data` 作为一个指针，指向的数组是一片连续的内存空间，这片内存空间可以用于存储切片中保存的全部元素，数组中的元素只是逻辑上的概念，底层存储其实都是连续的，所以我们可以将切片理解成一片连续的内存空间加上长度与容量的标识。

```go
mySlice := make([]int, 7)
```

`mySlice`的内存分布

![内存分布](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/pictures/slice-struct.png)

`mySlice`的`len`和`cap`都是`7`

从内存分布图我们可以看到，切片和数组的关系很密切，`Data`指向一个数组，切片引入了一个抽象层，提供了对数组中部分片段的引用，作为数组的引用，我们可以在运行区间可以修改它的长度，如果底层的数组长度不足就会触发扩容机制，切片中的数组就会发生变化，不过在上层看来切片时没有变化的，上层只需要与切片打交道不需要关心底层的数组变化。

### 切片常用的用法

#### 切片初始化

Go 语言中的切片有三种初始化的方式：

* 通过下标的方式获得数组或者切片的一部分；

    使用下标创建的切片与原数组或原切片共享底层的数组。

* 使用字面量初始化新的切片；

    字面量初始化新切片时，先创建一个数组，然后通过数组下标创建切片

* 使用关键字 `make` 创建切片：

    先申请一定容量的内存，创建数组， 新切片的`data`指向这个数组

```go
myArr[0:3]                  #数组切分
mySlice[0:3]                #切片切分
mySlice := []int{1, 2, 3}     #字面量定义
mySlice := make([]int, 10)    #make定义
```



#### 切片的追加和扩容

Go的切片通过`append`函数追加切片元素，追加到切片的后面，如果切片的容量不足时，会自动扩容，自动扩容时涉及到切片拷贝，也涉及到底层数组的拷贝；如果切片容量足够时，直接追加到切换后面。

* 当切片容量足够时直接插入

    ![直接插入](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/pictures/slice-insert.png)

* 当切片不足时，先要扩容拷贝，然后在插入

    ![扩容后插入](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/pictures/slice-grow.png)

    当扩容时，Go有一套扩容算法，大致规则如下：

    1. 如果期望容量大于当前容量的两倍就会使用期望容量；
    2. 如果当前切片容量小于 1024 就会将容量翻倍；
    3. 如果当前切片容量大于 1024 就会每次增加 25% 的容量，直到新容量大于期望容量；

#### 切片的遍历

可以通过`for ... range`对切片进行遍历操作

```go
for idx, val := range mySlice {
    fmt.Printf("idx: %d, val: %d\n", idx, val)
}
```



#### 切片的拷贝

可以通过`copy(mySlice, newSlice)`函数对切片进行拷贝，拷贝会将底层的数组也重新拷贝，可以理解是`深拷贝`，`mySlice`和`newSlice`的底层数组是相互独立的。

### 切片作为函数参数

Go函数的参数是按值传递，也是说函数参数是传入参数的一份拷贝，当切片作为函数参数时，会将切片参数的副本传入函数，但是切片和副本的底层数组是共享的，也就是说底层的数组没有拷贝，这个是`浅拷贝`，在函数中修改切片副本的数据，对传入的实参是有影响的。

```go
func main() {
    slice := []int{0, 1, 2, 3}
    fmt.Printf("slice: %v slice addr %p \n", slice, &slice)

    ret := changeSlice(slice)
    fmt.Printf("slice: %v ret: %v slice addr %p \n", slice, &slice, ret)
}

func changeSlice(s []int) []int {
    s[1] = 111
    return s
}
```

结果:

```go
slice: [0 1 2 3] slice addr 0xc00000c060
slice: [0 111 2 3] ret: &[0 111 2 3] slice addr 0xc000016120
```

从结果中可以看到，函数`changeSlice`修改了形参`s`，实参`slice`也发生了改变，形参的存储地址与实参不一致，可以说明，函数在参数传入时传入的是副本，但是底层数组是共享。

有种情况需要注意，当向形参`s`调用`append`函数追加元素时，有可能导致形参`s`的容量不足，产生拷贝，底层的数组也发生拷贝，形参`s`的底层数组跟实参`slice`的底层数组不在共享，就有可能导致实参`slice`没有被修改，看下面这个例子：

```go
package main

import "fmt"

func main() {
	slice := []int{0, 1, 2, 3}
	fmt.Printf("slice: %v slice addr %p \n", slice, &slice)

	ret := appendSlice(slice)
	fmt.Printf("slice: %v ret: %v slice addr %p \n", slice, ret, ret)
}

func appendSlice(s []int) []int {
	nSlice := []int{5, 6, 7, 8, 9, 10}

	s = append(s, nSlice...)

	return s
}
```

执行的结果：

```go
slice: [0 1 2 3] slice addr 0xc00000c060
slice: [0 1 2 3] ret: [0 1 2 3 5 6 7 8 9 10] slice addr 0xc000090000
```

实参`slice`还是原来的数据，但是形参`s`已经改变，这种情况需要注意，需要对切分的`append`深刻理解。

如果想通过修改形参`s`来修改实参`slice`的话，可以将函数`appendSlice`的参数类型修改为切片的指针类型`s *[]int`，看一下下面的代码：

```go
package main

import "fmt"

func main() {
	slice := []int{0, 1, 2, 3}
	fmt.Printf("slice: %v slice addr %p \n", slice, &slice)

	ret := appendSlice(&slice)
	fmt.Printf("slice: %v ret: %v slice addr %p \n", slice, ret, ret)
}

func appendSlice(s *[]int) []int {
	nSlice := []int{5, 6, 7, 8, 9, 10}

	*s = append(*s, nSlice...)

	return *s
}
```

执行结果：

```go
slice: [0 1 2 3] slice addr 0xc0000a0000
slice: [0 1 2 3 5 6 7 8 9 10] ret: [0 1 2 3 5 6 7 8 9 10] slice addr 0xc0000ae000
```

形参和实参都改变，而且存储地址都是一样的，说明形参和实参都是指向同一个切片

### nil切片和空切片

![nil切片和空切片内存分布](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/pictures/slice-nil.png)

* nil切片中指针是nil，len为0，cap也为0，只是定义切片，没有实例化

    ```go
    var myNilSlice []int #只定义没有初始化，这时myNilSlice为ni
    ```

* 空切片中指针指向空数组，len为0，cap也为0

    ```go
    myS := []int{} #通过字面量定义空切片
    myS := make([]int, 0) #通过make创建空切片
    ```

* `append`、`len`、`cap`对nil切片和空切片都是一样的

    ```go
    package main
    
    import "fmt"
    
    func main() {
        var mySliceNil []int
    	fmt.Printf("mySliceNil: %v, addr: %p, len %d, cap %d\n", mySliceNil, mySliceNil, len(mySliceNil), cap(mySliceNil))
    
    	//mySliceNil[0] = 1  #这样是错误的，会提示越界
    	mySliceNil = append(mySliceNil, 2)
    	fmt.Printf("mySliceNil: %v, addr: %p, len %d, cap %d\n", mySliceNil, mySliceNil, len(mySliceNil), cap(mySliceNil))
    
    	myS := []int{}
    	fmt.Printf("myS: %v, addr: %p, len %d, cap %d\n", myS, myS, len(myS), cap(myS))
    
        //myS[0] = 1  #这样是错误的，会提示越界
    	myS = append(myS, 2)
    	fmt.Printf("myS: %v, addr: %p, len %d, cap %d\n", myS, myS, len(myS), cap(myS))
    }
    ```

    指向结果：

    ```go
    mySliceNil: [], addr: 0x0, len 0, cap 0
    mySliceNil: [2], addr: 0xc0000160a0, len 1, cap 1
    myS: [], addr: 0x118ffd0, len 0, cap 0
    myS: [2], addr: 0xc0000160c0, len 1, cap 1
    ```



### 总结

* 切片是基于数组构建的，切片的数据结构是指向数组指向`data`、长度`len`、容量`cap`组成，其中`data`指向底层的数组。
* 切片是引用类型，默认值是`nil`，Go的数组是值语义。
* 切片操作并不复制切片指向的元素。它创建一个新的切片并复用原来切片的底层数组。 这使得切片操作和数组索引一样高效。同时，通过一个新切片修改元素会影响到原始切片的对应元素。可以通过`copy`函数复制需要的`slice`。
* 增加切片的容量实际上是创建一个新的、更大容量的切片，然后将原有切片的内容复制到新的切片。

### 引用

* [切片](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array-and-slice/)
* [[Go 切片：用法和本质](https://blog.csdn.net/AMimiDou_212/article/details/84451091)]
* [golang slice的技巧](https://github.com/golang/go/wiki/SliceTricks#filtering-without-allocating)
