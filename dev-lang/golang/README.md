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

2. [空结构体](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/golang%E7%A9%BA%E7%BB%93%E6%9E%84%E4%BD%93.md)

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
[包原理](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go%E5%8C%85%E5%8E%9F%E7%90%86.md)

## JSON
[JSON编解码](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go%E4%B8%ADjson%E7%BC%96%E8%A7%A3%E7%A0%81.md)

## mysql
[mysql](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go-mysql.md)

## 闭包
[闭包](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go%E9%97%AD%E5%8C%85%E4%B8%8E%E5%8C%BF%E5%90%8D%E5%87%BD%E6%95%B0.md)

## 单元测试
[GO语言单元测试入门](https://davidlovezoe.club/wordpress/archives/419)

## debug
* [GO语言开发调试入门](https://davidlovezoe.club/wordpress/archives/373)

* [GO语言开发调试高阶](https://davidlovezoe.club/wordpress/archives/386)

## 最佳实践
* [值传递和引用传递](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/%E5%80%BC%E4%BC%A0%E9%80%92%E5%92%8C%E5%BC%95%E7%94%A8%E4%BC%A0%E9%80%92.md)

* [50 tips](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/Go-50-tips.md)

* [理解nil](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/%E7%90%86%E8%A7%A3Go%E4%B8%ADnil.md)

* [理解go momory](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/go-memory.md)

* [rune类型](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/rune%E7%B1%BB%E5%9E%8B%E7%9A%84%E4%BD%BF%E7%94%A8%E4%B8%8E%E7%90%86%E8%A7%A3.md)

## golang不错的公众号和博客
* [go-grpc](https://github.com/lizj3624/mynote/blob/master/dev-lang/golang/grpc-go.md)

* [go package](https://golang.org/pkg/)

* [码农桃花源](https://qcrao91.gitbook.io/go/)

* [面向信仰编程](https://draveness.me/golang/)

* [Go语言中文网](https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MzAxMTA4Njc0OQ==&scene=124#wechat_redirect)

* [Go中国](https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MjM5OTcxMzE0MQ==&scene=124#wechat_redirect)

* [飞雪无情](https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MzI3MjU4Njk3Ng==&scene=124#wechat_redirect)

* [Go夜读](https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MzAwNTc3OTE5Mg==&scene=124#wechat_redirect)

* [Go学习之路](https://github.com/developer-learning/learning-golang)

* [Golang语言社区](https://cloud.tencent.com/developer/column/2170)

* [Go语言101](https://gfw.go101.org/article/101.html)

* [Go语言42章经](https://github.com/ffhelicopter/Go42/blob/master/SUMMARY.md)

* [Go语言优秀资源整理，为项目落地加速](https://shockerli.net/post/go-awesome/)

* [Golang源码解读](https://github.com/cch123/golang-notes)

## golang不错的书籍

* Go语言圣经

* Go语言程序设计

* 精通Go

* [Go语言设计与实现](https://draveness.me/golang/)
