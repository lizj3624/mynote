- [golang对JSON处理](#golang对json处理)
  - [golang对JSON的编码](#golang对json的编码)
  - [基本结构编码](#基本结构编码)
  - [复合结构编码](#复合结构编码)
  - [嵌套结构编码](#嵌套结构编码)
    - [空接口编码](#空接口编码)
    - [编码成json就是null](#编码成json就是null)
  - [Struct Tag字段重命名](#struct-tag字段重命名)
    - [横杆`-`忽略字段](#横杆-忽略字段)
      - [`omitempty`可选字段](#omitempty可选字段)
    - [`string`选项](#string选项)
  - [总结](#总结)
  - [golang对JSON的解码](#golang对json的解码)
  - [引用](#引用)
# golang对JSON处理
## golang对JSON的编码
golang提供了encoding/json的标准库用于编码json，大致需要两步：

1. 首先定义json结构体。
2. 使用 Marshal方法序列化。

定义结构体的时候，只有字段名是大写的，才会被编码到json当中。

## 基本结构编码

```go
type Account struct {
    Email string
    password string
    Money float64
}

func main() {
    account := Account{
        Email: "rsj217@gmail.com",
        password: "123456",
        Money: 100.5,
    }

    rs, err := json.Marshal(account)
    if err != nil{
        log.Fatalln(err)
    }

    fmt.Println(rs)
    fmt.Println(string(rs))
}
```

可以看到输出如下，Marshal方法接受一个空接口的参数，返回一个[]byte结构。小写命名的password字段没有被编码到json当中，生成的json结构字段和Account结构一致。

```go
[123 34 69 109 97 105 108 34 58 34 114 115 106 50 49 55 64 103 109 97 105 108 46 99 111 109 34 44 34 77 111 110 101 121 34 58 49 48 48 46 53 125]
{"Email":"rsj217@gmail.com","Money":100.5}
```

## 复合结构编码

相比字串，数字等基本数据结构，slice切片，map图则是复合结构。这些结构编码也类似。不过map的key必须是字串，而value必须是同一类型的数据。

```go
type User struct {
    Name    string
    Age     int
    Roles   []string
    Skill   map[string]float64
}

func main() {

    skill := make(map[string]float64)

    skill["python"] = 99.5
    skill["elixir"] = 90
    skill["ruby"] = 80.0

    user := User{
        Name:"rsj217",
        Age: 27,
        Roles: []string{"Owner", "Master"},
        Skill: skill,
    }

    rs, err := json.Marshal(user)
    if err != nil{
        log.Fatalln(err)
    }
    fmt.Println(string(rs))
}
```

输出：

```go
{
    "Name":"rsj217",
    "Age":27,
    "Roles":[
        "Owner",
        "Master"
    ],
    "Skill":{
        "elixir":90,
        "python":99.5,
        "ruby":80
    }
}
```

## 嵌套结构编码

slice和map可以匹配json的数组和对象，当前提是对象的value是同类型的情况。更通用的做法，对象的key可以是string，但是其值可以是多种结构。golang可以通过定义结构体实现这种构造。

```go
type User struct {
    Name    string
    Age     int
    Roles   []string
    Skill   map[string]float64
    Account Account
}

func main(){
    
    ...

    user := User{
        Name:"rsj217",
        Age: 27,
        Roles: []string{"Owner", "Master"},
        Skill: skill,
        Account:account,
    }
    ...
}
```

输出：

```go
{
    "Name":"rsj217",
    "Age":27,
    "Roles":[
        "Owner",
        "Master"
    ],
    "Skill":{
        "elixir":90,
        "python":99.5,
        "ruby":80
    },
    "Account":{
        "Email":"rsj217@gmail.com",
        "Money":100.5
    }
}
```

### 空接口编码

通过定义嵌套的结构体Account，实现了key与value不一样的结构。golang的数组或切片，其类型也是一样的，如果遇到不同数据类型的数组，则需要借助空结构来实现：

```go
type User struct {
    ...

    Extra []interface{}
}

extra := []interface{}{123, "hello world"}

user := User{
    ...
    
    Extra:   extra,
}
```

输出：

```go
{
    ...
    "Extra":[
        123,
        "hello world"
    ]
}
```

使用空接口，也可以定义像结构体实现那种不同value类型的字典结构。当空接口没有初始化其值的时候，零值是 nil。

### 编码成json就是null

```go
type User struct {
    Name    string
    Age     int
    Roles   []string
    Skill   map[string]float64
    Account Account

    Extra []interface{}

    Level map[string]interface{}
}

func main() {

    ...

    level := make(map[string]interface{})

    level["web"] = "Good"
    level["server"] = 90
    level["tool"] = nil

    user := User{
        Name:    "rsj217",
        Age:     27,
        Roles:   []string{"Owner", "Master"},
        Skill:   skill,
        Account: account,
        Level:   level,
    }

    ...
}
```

输出：

```go
{
    ...
    "Extra":null,
    "Level":{
        "server":90,
        "tool":null,
        "web":"Good"
    }
}
```

可以看到 Extra返回的并不是一个空的切片，而是null。同时Level字段实现了向字典的嵌套结构.

## Struct Tag字段重命名

golang提供了`struct tag`的方式可以重命名结构字段的输出形式。

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"pass_word"`
    Money    float64 `json:"money"`
}

func main() {
    account := Account{
        Email:    "rsj217@gmail.com",
        Password: "123456",
        Money:    100.5,
    }

    rs, err := json.Marshal(account)
    ...
}
```

我们使用struct tag，重新给Account结构的字段进行了重命名。其中email小写了，并且password字段还使用了下划线，输出的结果如下：

```go
{"email":"rsj217@gmail.com","pass_word":"123456","money":100.5}
```

### 横杆`-`忽略字段

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"-"`
    Money    float64 `json:"money"`
}
```

输出：

```json
{"email":"rsj217@gmail.com","money":100.5}
```

`Password`被标记`-`，就没有在`json`中输出

#### `omitempty`可选字段

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password,omitempty"`
    Money    float64 `json:"money"`
}

func main() {
    account := Account{
        Email:    "rsj217@gmail.com",
        Password: "",
        Money:    100.5,
    }

    rs, err := json.Marshal(account)
    if err != nil {
        log.Fatalln(err)
    }

    fmt.Println(rs)
    fmt.Println(string(rs))
}
```

输出:

```json
{"email":"rsj217@gmail.com","money":100.5}
```

### `string`选项

golang是静态类型语言，对于类型定义的是不能动态修改。在json处理当中，struct tag的string可以起到部分动态类型的效果。有时候输出的json希望是数字的字符串，而定义的字段是数字类型，那么就可以使用string选项。

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password,omitempty"`
    Money    float64 `json:"money,string"`
}
func main() {
    account := Account{
        Email:    "rsj217@gmail.com",
        Password: "123",
        Money:    100.50,
    }

    ...
}
```

输出：

```json
{"email":"rsj217@gmail.com","money":"100.5"}
```

可以看到输出为 `money: "100.5"`， money字段输出的值是字串，但是定义结构体中是`float64`

## 总结

总体原则分两步，首先定义需要编码的结构，然后调用encoding/json标准库的Marshal方法生成json byte数组，转换成string类型即可。

* 定义对应的结构体
* 调用`func Marshal(v interface{}) ([]byte, error)`

## golang对JSON的解码

与编码`json`的`Marshal`类似，`Golang`解析`json`也提供了`Unmarshal`方法。对于解析`json`，也大致分两步，首先定义结构，然后调用Unmarshal方法序列化。

* 定义相应的结构体
* 调用`func Unmarshal(data []byte, v interface{}) error`，`Unmarshal`接受一个`byte`数组和空接口指针的参数。

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password"`
    Money    float64 `json:"money"`
}

var jsonString string = `{
    "email":"rsj217@gmail.com",
    "password":"123",
    "money":100.5
}`

func main() {

    account := Account{}

    err := json.Unmarshal([]byte(jsonString), &account)
    if err != nil{
        log.Fatalln(err)
    }
    fmt.Printf("%+v\n", account)
}
```

golang会将json的数据结构和go的数据结构进行匹配。匹配的原则就是寻找tag的相同的字段，然后查找字段。查询的时候是大小写不敏感的，小写字段不会被解析。

```go
type Account struct {
    Email    string  `json:"email"`
    PassWord string  
    Money    float64 `json:"money"`
}
```

输出`{Email:rsj217@gmail.com PassWord:123 Money:100.5}`，把 Password的tag去掉，再修改成PassWord，依然可以把json的password匹配到PassWord，但是如果结构的字段是私有的，即使tag符合，也不会被解析：

```go
type Account struct {
    Email    string  `json:"email"`
    password string  `json:"password"`
    Money    float64 `json:"money"`
}
```

输出`{Email:rsj217@gmail.com password: Money:100.5}`。上面的password并不会被解析赋值json的password，大小写不敏感只是针对公有字段而言。再寻找tag或字段的时候匹配不成功，则会抛弃这个json字段的值：

```go
type Account struct {
    Email    string  `json:"email"`
    Password string  `json:"password"`
}
```

输出 `{Email:rsj217@gmail.com Password:"123"}`， 并不会有money字段被赋值。

解码是也是支持在编码部分的`Struct Tge`

## 引用

* [Golang处理JSON（一）--- 编码](https://www.jianshu.com/p/f3c2105bd06b)
* [Golang处理JSON（二）--- 解码](https://www.jianshu.com/p/31757e530144)
* [[JSON and Go](https://blog.golang.org/json)]
* [JSON and Go中文](https://sanyuesha.com/2018/05/07/go-json/)
* [Go struct tag深入理解](https://my.oschina.net/renhc/blog/2045683)
* [StructTag](https://golang.org/pkg/reflect/#StructTag)

