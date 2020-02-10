## Go的字符串类型

字符串是Go中基础的数据类型之一，它是一段连续内存存储的，其实Go语言中的字符串其实是一个只读的字节数组。

![Go字符串存储](/Users/lizunju/workspace/github/mynote/golang/pictures/2019-12-31-15777265631608-in-memory-string.png)

图片引用[这里](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-string/)

### Go字符串的内部数据结构

```go
type StringHeader struct {
	Data uintptr
	Len  int
}
```

### 字符串定义

* 双引号定义

```go
var myStr = "Hello World"
myStr := "Hello World"
```

1. 标准字符串使用双引号表示开头和结尾；
2. 标准字符串中需要使用反斜杠 `\` 来 `escape` 双引号；
3. 标准字符串中不能出现如下所示的隐式换行符号 `\n`；

* 反引号定义

如果字符串中有包含双引号或者字符串有多行时，可用反引号``

```go
myStr := `Hello 
World`

myJson := `{"name":"Brace", "age": 23}`
```

### 字符串的拼接

Go字符串主要通过`+`进行拼接

```go
myStr ：= "Hello World"
myStr = myStr + "!!!"
```

字符串变量是不可以修改，`myStr = myStr + "!!!"`是`myStr + "!!!"`先生成新字符串，然后新字符串再给`myStr`赋值。

### 字符串与[]byte转换

字符串和 `[]byte` 中的内容虽然一样，但是字符串的内容是只读的，我们不能通过下标或者其他形式改变其中的数据，而 `[]byte` 中的内容是可以读写的，无论从哪种类型转换到另一种都需要对其中的内容进行拷贝，而内存拷贝的性能损耗会随着字符串和 `[]byte` 长度的增长而增长。