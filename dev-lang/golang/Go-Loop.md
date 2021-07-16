## Go中的循环

Go中没有`while`和`do...while`的循环语句，使用`for`循环

### 标准的for循环

* for的死循环

```go
##读取文件内容
fp, err := os.Open(f)
fReader := bufio.NewReader(fp)
for {
    line, err := fReader.ReadString('\n')
    if err != nil {
        if err == io.EOF {
            break
        }
        break
     }
    ...
}
```

* for的固定循环

```go
for i := 0; i < 10; i++ {
    ...
}
```

### for-range循环

在Go中`for-range`循环遍历比较常见，它可以遍历数组、切片、map、channel。

```go
### slice
s := make([]string, 0, 10)
s = apppend(a, "hello")
for k, v := range s {
    fmt.Printf("k: %d, v: %s\n", k, v)
}

### map，遍历是无序的
m := make(map[string]int, 10)
m["1"] = 1
for k, v := range m {
    fmt.Printf("k: %s, v: %d\n", k, v)
}

### channel
queue := make(chan string, 2)
queue <- "one"
queue <- "two"
close(queue)   ###优雅关闭
for elem := range queue {
    fmt.Println(elem)
}
```

`for-range`遍历时发生copy，`k，v`是遍历数据的副本。

注意的`k，v`是副本的问题

```go
func main() {
	arr := []int{1, 2, 3}
	newArr := []*int{}
	for _, v := range arr {
		newArr = append(newArr, &v)
	}
	for _, v := range newArr {
		fmt.Println(*v)
	}
}

$ go run main.go
3 3 3
```

输出结构都是`3`，是因为`v`是副本，`&v`只是变量的地址，遍历时每次给变量`v`赋值，因此变量`v`是最后`3`的值，将`&v`修改`&arr[i]`

