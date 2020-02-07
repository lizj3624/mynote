## GO Map详解

Go Map是`k/v`的数据结构，内部主要是通过`Hash`实现的，`Map`的`key`可以是**内置类型**，也可以是**结构类型**，只要可以使用 `==` 运算符做比较，都可以作为`key`。**切片**、**函数**以及**包含切片的结构类型**，这些类型由于具有引用语义，不能作为`key`，使用这些类型会造成编译错误。

### Map的数据结构

Map的底层数据结构，如下引用"理解Golang哈希表Map的原理"

```go
type hmap struct {
    count     int
    flags     uint8
    B         uint8
    noverflow uint16
    hash0     uint32

    buckets    unsafe.Pointer
    oldbuckets unsafe.Pointer
    nevacuate  uintptr

    extra *mapextra
}
```

1. `count` 表示当前哈希表中的元素数量；
2. `B` 表示当前哈希表持有的 `buckets` 数量，但是因为哈希表中桶的数量都 2 的倍数，所以该字段会存储对数，也就是 `len(buckets) == 2^B`；
3. `hash0` 是哈希的种子，它能为哈希函数的结果引入随机性，这个值在创建哈希表时确定，并在调用哈希函数时作为参数传入；
4. `oldbuckets` 是哈希在扩容时用于保存之前 `buckets` 的字段，它的大小是当前 `buckets` 的一半；

![Map内存数据结构图](https://github.com/lizj3624/mynote/blob/master/golang/pictures/2019-12-30-15777168478811-hmap-and-buckets.png)

![Map详细数据结构图](https://github.com/lizj3624/mynote/blob/master/golang/pictures/1jjbeg6th6.jpeg)

### Map的创建

#### 字面量定义

```go
hash := map[string]int{
	"1": 2,
	"3": 4,
	"5": 6,
}
```

#### make关键字定义

```go
hash := make(map[string]int, 3)
hash["1"] = 2
hash["3"] = 4
hash["5"] = 6
```

### Map的使用

map有插入、删除、更新等操作，但是这些操作不是协程/线程安全的，需要对临界区进行保护

```go
hash := make(map[string]int, 3)
### 插入
hash["4"] = 3
hash["5"] = 6

### 更新
hash["4"] = 4

### 删除
delete(hash, "4")
```

### 引用

* [深度解密Go语言之map](https://mp.weixin.qq.com/s/2CDpE5wfoiNXm1agMAq4wA)
* [理解Golang哈希表Map的原理](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-hashmap/)
* [深入理解Go map：初始化和访问元素](https://cloud.tencent.com/developer/article/1422446)
* [Go Map -- 就要学习 Go 语言](https://juejin.im/post/5c446a9af265da616d5475a3)