- [Golang](#golang)
  - [简单语法](#简单语法)
  - [string](#string)
  - [slice](#slice)
  - [map](#map)
  - [channel](#channel)
  - [结构体struct](#结构体struct)
  - [context包](#context包)
  - [接口](#接口)
  - [反射](#反射)
  - [go-module](#go-module)
  - [defer](#defer)
  - [正则表达式](#正则表达式)
  - [package](#package)
  - [JSON](#json)
  - [mysql](#mysql)
  - [单元测试](#单元测试)
  - [debug](#debug)
  - [最佳实践](#最佳实践)
  - [golang不错的公众号和博客](#golang不错的公众号和博客)
# Golang
## 简单语法
1. go的源文件以`.go`结尾，以**小写**字母组成，比如: `scanner.go`, 多个单词可以通过下划线`_`组成，比如: `scanner_test.go`    
   
2. 包名用小写字母组成, `package service`，一个包的源文件放到与包名相同的文件夹中，比如: `package service`的源文件放到`service`目录下。通过`import`引入包，一个包只引用一次，每段代码也只编译一次。    

3. 基本类型: `int float bool string`，零值：`int 0, float 0.0, bool false, string ""`；复合类型： `struct array slice map channel`，零值: `nil`；描述行为: `interface`，

4. 常量类型只能是bool，数字，字符串
```go
// const identifier [type] = value
const Pi = 3.14159

const (
    a = iota  //0
    b
    c
)
```
5. 变量区分大小写，var声明变量后，变量被初始化零值，变量名称按照驼峰命名法，首字符小写
```go
var i int

var i int = 1

i := 1
```
6. 位运算: 与 `&`,  或`|`, 异或`^`，左移`<<`，右移`>>`

7. 控制结构
```go
// if else
if condition {
    //
} else if condition {
    //
} else {
    //
}

//switch
switch var1 {
    case val1: 
        //
    case val2:
        //
    default:
        //
}

//for
for {
    //
}

for i := 0; i < 5; i++ {
    //
}

for _, i := range slice01 {
    //
}
```
[for循环用法](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/for-loop.md)

8. golang代码实例
```go
package main

import (
    "fmt"
)

const c = "C"

var v int = 5

type T struct {
    //
}

func main() {
    var a int
    Func1()
    //...
    fmt.Println(a)
}

func (t T) Method1() {
    //
}

func Func1() {
    //
}
```

## string

## slice
* [slice](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/slice.md)
* [slice参数时的陷阱](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go-slice-function-paramter-trap.md)

## map
[map](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/map.md)

## channel
[channel](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/channel.md)

## 结构体struct
1. [struct](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go-struct.md)
2. [空结构体]()

## context包
[map](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/context.md)

## 接口
[interface](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/interface.md)

## 反射
[反射](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go-reflect.md)

## go-module
[module](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go-module.md)

## defer
[defer](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/defer.md)

## 正则表达式
[正则](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go-regexp.md)

## package

## JSON

## mysql
[mysql](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go-mysql.md)

## 单元测试
[[GO语言单元测试入门](https://davidlovezoe.club/wordpress/archives/419)]

## debug
* [[GO语言开发调试入门](https://davidlovezoe.club/wordpress/archives/373)]
* [[GO语言开发调试高阶](https://davidlovezoe.club/wordpress/archives/386)]

## 最佳实践
* [值传递和引用传递]()
* [50 tips]()
* [理解nil]()
* [理解go momory]()

## golang不错的公众号和博客
* [golang不错的公众号和博客]()
* [go-grpc](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/grpc-go.md)
