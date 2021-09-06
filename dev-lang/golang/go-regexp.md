- [正则表达式](#正则表达式)
  - [正则表达式语法规则](#正则表达式语法规则)
    - [1) 字符](#1-字符)
    - [2) 数量词（用在字符或 (...) 之后）](#2-数量词用在字符或--之后)
    - [3) 边界匹配](#3-边界匹配)
    - [4) 逻辑、分组](#4-逻辑分组)
    - [5) 特殊构造（不作为分组）](#5-特殊构造不作为分组)
  - [Regexp 包的使用](#regexp-包的使用)
    - [匹配指定类型的字符串](#匹配指定类型的字符串)
    - [匹配a和c中间包含一个数字的字符串](#匹配a和c中间包含一个数字的字符串)
    - [使用 `\d` 来匹配a和c中间包含一个数字的字符串。](#使用-d-来匹配a和c中间包含一个数字的字符串)
    - [匹配字符串中的小数](#匹配字符串中的小数)
    - [匹配`div`标签中的内容](#匹配div标签中的内容)
    - [通过`Compile`方法返回一个`Regexp`对象，实现匹配，查找，替换相关的功能](#通过compile方法返回一个regexp对象实现匹配查找替换相关的功能)
# 正则表达式
正则表达式是一种进行模式匹配和文本操纵的复杂而又强大的工具。虽然正则表达式比纯粹的文本匹配效率低，但是它却更灵活，按照它的语法规则，根据需求构造出的正则表达式能够从原始文本中筛选出几乎任何你想要得到的字符组合。

Go语言通过 regexp 包为正则表达式提供了官方支持，其采用 RE2 语法，除了`\c`、`\C`外，Go语言和 Perl、[Python](http://c.biancheng.net/python/) 等语言的正则基本一致。

## 正则表达式语法规则

正则表达式是由普通字符（例如字符 a 到 z）以及特殊字符（称为"元字符"）构成的文字序列，可以是单个的字符、字符集合、字符范围、字符间的选择或者所有这些组件的任意组合。

下面的表格中列举了构成正则表达式的一些语法规则及其含义。

### 1) 字符

| 语法     | 说明                                                         | 表达式示例 | 匹配结果          |
| -------- | ------------------------------------------------------------ | ---------- | ----------------- |
| 一般字符 | 匹配自身                                                     | abc        | abc               |
| .        | 匹配任意除换行符"\n"外的字符， 在 DOTALL 模式中也能匹配换行符 | a.c        | abc               |
| \        | 转义字符，使后一个字符改变原来的意思； 如果字符串中有字符 * 需要匹配，可以使用 \* 或者字符集［*]。 | a\.c a\\c  | a.c a\c           |
| [...]    | 字符集（字符类），对应的位置可以是字符集中任意字符。 字符集中的字符可以逐个列出，也可以给出范围，如 [abc] 或 [a-c]， 第一个字符如果是 ^ 则表示取反，如 [^abc] 表示除了abc之外的其他字符。 | a[bcd]e    | abe 或 ace 或 ade |
| \d       | 数字：[0-9]                                                  | a\dc       | a1c               |
| \D       | 非数字：[^\d]                                                | a\Dc       | abc               |
| \s       | 空白字符：[<空格>\t\r\n\f\v]                                 | a\sc       | a c               |
| \S       | 非空白字符：[^\s]                                            | a\Sc       | abc               |
| \w       | 单词字符：[A-Za-z0-9]                                        | a\wc       | abc               |
| \W       | 非单词字符：[^\w]                                            | a\Wc       | a c               |

### 2) 数量词（用在字符或 (...) 之后）

| 语法  | 说明                                                         | 表达式示例 | 匹配结果     |
| ----- | ------------------------------------------------------------ | ---------- | ------------ |
| *     | 匹配前一个字符 0 或无限次                                    | abc*       | ab 或 abccc  |
| +     | 匹配前一个字符 1 次或无限次                                  | abc+       | abc 或 abccc |
| ?     | 匹配前一个字符 0 次或 1 次                                   | abc?       | ab 或 abc    |
| {m}   | 匹配前一个字符 m 次                                          | ab{2}c     | abbc         |
| {m,n} | 匹配前一个字符 m 至 n 次，m 和 n 可以省略，若省略 m，则匹配 0 至 n 次； 若省略 n，则匹配 m 至无限次 | ab{1,2}c   | abc 或 abbc  |

### 3) 边界匹配

| 语法 | 说明                                         | 表达式示例 | 匹配结果 |
| ---- | -------------------------------------------- | ---------- | -------- |
| ^    | 匹配字符串开头，在多行模式中匹配每一行的开头 | ^abc       | abc      |
| $    | 匹配字符串末尾，在多行模式中匹配每一行的末尾 | abc$       | abc      |
| \A   | 仅匹配字符串开头                             | \Aabc      | abc      |
| \Z   | 仅匹配字符串末尾                             | abc\Z      | abc      |
| \b   | 匹配 \w 和 \W 之间                           | a\b!bc     | a!bc     |
| \B   | [^\b]                                        | a\Bbc      | abc      |

### 4) 逻辑、分组

| 语法          | 说明                                                         | 表达式示例           | 匹配结果       |
| ------------- | ------------------------------------------------------------ | -------------------- | -------------- |
| \|            | \| 代表左右表达式任意匹配一个，优先匹配左边的表达式          | abc\|def             | abc 或 def     |
| (...)         | 括起来的表达式将作为分组，分组将作为一个整体，可以后接数量词 | (abc){2}             | abcabc         |
| (?P<name>...) | 分组，功能与 (...) 相同，但会指定一个额外的别名              | (?P<id>abc){2}       | abcabc         |
| \<number>     | 引用编号为 <number> 的分组匹配到的字符串                     | (\d)abc\1            | 1abe1 或 5abc5 |
| (?P=name)     | 引用别名为 <name> 的分组匹配到的字符串                       | (?P<id>\d)abc(?P=id) | 1abe1 或 5abc5 |

### 5) 特殊构造（不作为分组）

| 语法      | 说明                                                         | 表达式示例        | 匹配结果         |
| --------- | ------------------------------------------------------------ | ----------------- | ---------------- |
| (?:...)   | (…) 的不分组版本，用于使用 "\|" 或后接数量词                 | (?:abc){2}        | abcabc           |
| (?iLmsux) | iLmsux 中的每个字符代表一种匹配模式，只能用在正则表达式的开头，可选多个 | (?i)abc           | AbC              |
| (?#...)   | # 后的内容将作为注释被忽略。                                 | abc(?#comment)123 | abc123           |
| (?=...)   | 之后的字符串内容需要匹配表达式才能成功匹配                   | a(?=\d)           | 后面是数字的 a   |
| (?!...)   | 之后的字符串内容需要不匹配表达式才能成功匹配                 | a(?!\d)           | 后面不是数字的 a |
| (?<=...)  | 之前的字符串内容需要匹配表达式才能成功匹配                   | (?<=\d)a          | 前面是数字的a    |
| (?<!...)  | 之前的字符串内容需要不匹配表达式才能成功匹配                 | (?<!\d)a          | 前面不是数字的a  |

## Regexp 包的使用

几个函数方法

| 名称                  | 说明                                                         | 备注 |
| --------------------- | ------------------------------------------------------------ | ---- |
| Match                 | 验证正则表达式是否匹配 []byte                                | -    |
| MatchString           | 验证正则表达式是否匹配 string                                | -    |
| FindAllString         | Regexp的方法，匹配字符串，返回匹配结果组成一个 []string。限定参数 -1表示不限定，其它表示限定。 | -    |
| FindAllStringSubmatch | Regexp的方法，返回一个 [][]string                            | -    |

下面通过几个示例来演示一下 regexp 包的使用。

### 匹配指定类型的字符串

```golang
package main

import (
    "fmt"
    "regexp"
)

func main() {

    buf := "abc azc a7c aac 888 a9c  tac"

    //解析正则表达式，如果成功返回解释器
    reg1 := regexp.MustCompile(`a.c`)
    if reg1 == nil {
        fmt.Println("regexp err")
        return
    }

    //根据规则提取关键信息
    result1 := reg1.FindAllStringSubmatch(buf, -1)
    fmt.Println("result1 = ", result1)
}
```

运行结果如下：

```golang
result1 = [[abc] [azc] [a7c] [aac] [a9c]]　　
```

### 匹配a和c中间包含一个数字的字符串

```golang
package main

import (
    "fmt"
    "regexp"
)

func main() {

    buf := "abc azc a7c aac 888 a9c  tac"

    //解析正则表达式，如果成功返回解释器
    reg1 := regexp.MustCompile(`a[0-9]c`)

    if reg1 == nil { //解释失败，返回nil
        fmt.Println("regexp err")
        return
    }

    //根据规则提取关键信息
    result1 := reg1.FindAllStringSubmatch(buf, -1)
    fmt.Println("result1 = ", result1)
}
```

运行结果如下：

```go
result1 = [[a7c] [a9c]]
```

### 使用 `\d` 来匹配a和c中间包含一个数字的字符串。

```golang
package main

import (
    "fmt"
    "regexp"
)

func main() {

    buf := "abc azc a7c aac 888 a9c  tac"

    //解析正则表达式，如果成功返回解释器
    reg1 := regexp.MustCompile(`a\dc`)
    if reg1 == nil { //解释失败，返回nil
        fmt.Println("regexp err")
        return
    }

    //根据规则提取关键信息
    result1 := reg1.FindAllStringSubmatch(buf, -1)
    fmt.Println("result1 = ", result1)
}
```

运行结果如下：

```golang
result1 = [[a7c] [a9c]]
```

### 匹配字符串中的小数

```golang
package main

import (
    "fmt"
    "regexp"
)

func main() {
    buf := "43.14 567 agsdg 1.23 7. 8.9 1sdljgl 6.66 7.8   "

    //解释正则表达式
    reg := regexp.MustCompile(`\d+\.\d+`)
    if reg == nil {
        fmt.Println("MustCompile err")
        return
    }

    //提取关键信息
    //result := reg.FindAllString(buf, -1)
    result := reg.FindAllStringSubmatch(buf, -1)
    fmt.Println("result = ", result)
}
```

运行结果如下：

```go
result = [[43.14] [1.23] [8.9] [6.66] [7.8]]
```

### 匹配`div`标签中的内容

```go
package main

import (
    "fmt"
    "regexp"
)

func main() {
    // 原生字符串
    buf := `
    
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <title>C语言中文网 | Go语言入门教程</title>
</head>
<body>
    <div>Go语言简介</div>
    <div>Go语言基本语法
    Go语言变量的声明
    Go语言教程简明版
    </div>
    <div>Go语言容器</div>
    <div>Go语言函数</div>
</body>
</html>
    `

    //解释正则表达式
    reg := regexp.MustCompile(`<div>(?s:(.*?))</div>`)
    if reg == nil {
        fmt.Println("MustCompile err")
        return
    }

    //提取关键信息
    result := reg.FindAllStringSubmatch(buf, -1)

    //过滤<></>
    for _, text := range result {
        fmt.Println("text[1] = ", text[1])
    }
}
```

运行结果如下：

```go
text[1] = Go语言简介
text[1] = Go语言基本语法
  Go语言变量的声明
  Go语言教程简明版

text[1] = Go语言容器
text[1] = Go语言函数
```

### 通过`Compile`方法返回一个`Regexp`对象，实现匹配，查找，替换相关的功能

```golang
package main
import (
    "fmt"
    "regexp"
    "strconv"
)
func main() {
    //目标字符串
    searchIn := "John: 2578.34 William: 4567.23 Steve: 5632.18"
    pat := "[0-9]+.[0-9]+"          //正则

    f := func(s string) string{
        v, _ := strconv.ParseFloat(s, 32)
        return strconv.FormatFloat(v * 2, 'f', 2, 32)
    }
    if ok, _ := regexp.Match(pat, []byte(searchIn)); ok {
        fmt.Println("Match Found!")
    }
    re, _ := regexp.Compile(pat)
    //将匹配到的部分替换为 "##.#"
    str := re.ReplaceAllString(searchIn, "##.#")
    fmt.Println(str)
    //参数为函数时
    str2 := re.ReplaceAllStringFunc(searchIn, f)
    fmt.Println(str2)
}
```

输出结果：
```go
Match Found!
John: ##.# William: ##.# Steve: ##.#
John: 5156.68 William: 9134.46 Steve: 11264.36
```
上面代码中 Compile 方法可以解析并返回一个正则表达式，如果成功返回，则说明该正则表达式正确可用于匹配文本。另外我们也可以使用 MustCompile 方法，它也可以像 Compile 方法一样检验正则的有效性，但是当正则不合法时程序将 panic。