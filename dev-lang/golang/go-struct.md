- [struct](#struct)
  - [定义结构体](#定义结构体)
  - [声明结构体](#声明结构体)
    - [new声明结构体指针变量](#new声明结构体指针变量)
    - [字面量声明结构体](#字面量声明结构体)
    - [选择器(selector)](#选择器selector)
    - [匿名字段和内嵌结构体](#匿名字段和内嵌结构体)
      - [匿名字段](#匿名字段)
      - [内嵌结构体](#内嵌结构体)
  - [结构体标签](#结构体标签)
  - [方法](#方法)
    - [指针或值作为接受者](#指针或值作为接受者)
  - [引用](#引用)
# struct
结构体由一系列命名的元素组成，这些元素又被称为字段，每个字段都有一个名称和一个类型。
结构体的目的就是把数据聚集在一起，以便能够更加便捷地操作这些数据。结构体的概念在 C 语言里很常见，被称为`struct`。`Golang`中的结构体也是`struct`。**`Go`语言中没有类的概念，因此在`Go`中结构体有着更为重要的地位。**结构体是复合类型(composite types)，当需要定义一个类型，它由一系列属性组成，每个属性都有自己的类型和值的时候，就应该使用结构体，它把数据聚集在一起。然后可以访问这些数据，就好像它是一个独立实体的一部分。**结构体也是值类型，因此可以通过`new`函数来创建。**

## 定义结构体

在`Golang`中最常用的方法是使用关键字`type`和`struct`来定义一个结构体，以关键字`type`开始，之后是新类型的名字，最后是关键字`struct`：

```go
// Person 为用户定义的一个类型
type Person struct {
    Name  string
    Age     int
    Email string
}
```

结构体中的字段都有名字，例如`Name`、`Age`、`Email`，如果名字在代码中从来不会被用到时，可以用`_`，结构体中的字段可以使任何类型，甚至是结构体本身，或者函数或者接口。

## 声明结构体

### new声明结构体指针变量

通过关键字`new`声明一个结构体变量指针，同时给这个变量分配内存，结构体内部的字段会自动被初始化相应类型的零值，`var t *T = new(T) `

```go
var p *Person = new(Person)

p := new(Person)
```

### 字面量声明结构体

```go
var man Person  //Person类型的变量man，其中字段自动会被初始化相应类型的零值
```

```go
//使用字面量定义，并对其初始化
var man Person = Person{
    Name: "Jack",
    Age: 30,
    Email: "jack@gmail.com",
}

women := &Person{
    Name: "Rose",
    Age: 32,
    Email: "rose@gmail.com",  //分号不能少
} 

we := &Peson{"Ketty", 23, "ketty@gmail.com"}
```

**表达式 new(Type) 和 &Type{} 是等价的。**

**结构体中字段如果有`slice`、`map`、`channel`时，声明结构体变量后，还需要对其进行`make`操作**

### 选择器(selector)

在`Golang`中，访问结构体成员需要使用点号操作符，点号操作符也被称为选择器(`selector`)，使用时的格式为：

```undefined
结构体.成员名
```

注意：这里的 "结构体" 是指结构体类型的变量或结构体指针类型的变量。无论变量是一个结构体类型还是一个结构体类型指针，都可以使用相同的选择器服务来引用结构体的字段。

### 匿名字段和内嵌结构体

结构体可以包含一个或多个匿名(或者称为内嵌)字段，即这些字段没有显式的名字。仅指明字段的类型，此时该类型就是字段的名字。匿名字段本身可以是一个结构体类型，即结构体可以包含内嵌的结构体。

#### 匿名字段
匿名字段和面向对象编程中的继承概念相似，可以被用来模拟类似继承的行为。`Golang`中的基础就是通过内嵌或组合来实现的，所以说在`Golang`中组合比继承更受欢迎。比如下面的例子：

```go
type Person struct {
    Name string
    Age int
    Email string
    int // 匿名字段
}

var p Person

p.int  //访问匿名字段
```

**在结构体中同一种类型的匿名字段只能有一个。**

#### 内嵌结构体

结构体也是一种数据类型，所以它同样可以作为匿名字段使用：

```go
type Person struct {
    Name  string
    Age     int
    Email string
}
type Student struct {
    Person
    StudentID int
}
```

下面的代码可以声明并初始化`Student`类型的变量：

```go
st := Student {
    Person:    Person{"jack", 22, "jack@xxx.com"},
    StudentID: 1000,
}
```

从这个示例可以看出，内层结构体被简单地插入或者内嵌进外层结构体。这种简单的**继承**机制使得`Golang`很轻松就能实现从一个或一些类型中继承部分或全部的实现。



## 结构体标签

```go
// Person 为用户定义的一个类型，带标签
type Person struct {
    Name  string    `json:"name`
    Age     int     `json:"age"`
    Email string    `json:"e-mail"`
}
```

结构体处理名称和类型外，还有一个可选的标签`tag`，它附属于字段的字符串，可以使文档或者其他重要标记。标签的内容一般不再编程中使用，只有包`refect`能获取它。目前用的比较多`tag`的地方是`json`编解码时指定相应`json`的'key'的名字。

## 方法

`Golang`语言的结构体很像面向对象编程中的**类**，`Golang`结构体中字段像是**类**的成员变量，`Golang`结构体中的方法就像**类**的方法，但是`Golang`的方法是作用在接受者`receiver`上的一个函数，接受者就是结构体类型的变量，因此方法是一种特殊类型的函数。

接受者可以是任何类型，甚至是函数，但是接受者不能是一个接口类型，结构体和方法可以不再同一个文件中，但是必须在同一个包内。

```go
func (p *Person) YourName() {
    fmt.Printf(p.Name)
}

func (p Person) YourAge() {
    fmt.Printf(p.Age)
}
```



### 指针或值作为接受者

* 指针作为接受者时，方法可以改变接受者的数据
* 值作为接受者时，方法只改变了接受者副本的数据，接受者本身没有变
* 指针方法和非指针方法都可以在指针或者非指针上调用

```go
func (p *Person) SetName(name string) {
    p.Name = name              //改变接受者p.Name的值
}

func (p Person) SetAge(age int) {
    p.Age = age               //只修改副本
}
```

## 引用

* [Golang 入门 : 结构体(struct)](https://www.jianshu.com/p/38e68458841b?utm_campaign=studygolang.com&utm_medium=studygolang.com&utm_source=studygolang.com)

* [the way to go](https://github.com/unknwon/the-way-to-go_ZH_CN)
* [struct的内存对齐](https://mp.weixin.qq.com/s/qPILuArUBnNrJ15COpBziQ)

