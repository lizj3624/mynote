- [mysql](#mysql)
  - [Mysql的连接与连接池](#mysql的连接与连接池)
    - [database/sql](#databasesql)
    - [Go的Mysql库](#go的mysql库)
    - [sql.DB](#sqldb)
    - [连接池](#连接池)
      - [连接失败](#连接失败)
      - [连接池配置](#连接池配置)
  - [CURD基础操作](#curd基础操作)
    - [读取数据](#读取数据)
    - [读取单条记录](#读取单条记录)
    - [rows.Scan原理](#rowsscan原理)
    - [空值处理](#空值处理)
    - [自动匹配字段](#自动匹配字段)
    - [Exec](#exec)
    - [总结](#总结)
  - [Prepared剖析](#prepared剖析)
    - [mysql的sql处理过程](#mysql的sql处理过程)
      - [即时`SQL`](#即时sql)
      - [预处理`SQL`](#预处理sql)
    - [自定义`prepare`查询](#自定义prepare查询)
    - [总结](#总结-1)
  - [事物](#事物)
    - [Transaction 事务](#transaction-事务)
    - [tx对象](#tx对象)
    - [事务与连接](#事务与连接)
    - [事务并发](#事务并发)
    - [实践](#实践)
    - [总结](#总结-2)
  - [引用](#引用)
# mysql
## Mysql的连接与连接池

### database/sql

`database/sql`是`golang`的标准库之一，它提供了一系列接口方法，用于访问关系数据库。它并不会提供数据库特有的方法，那些特有的方法交给数据库驱动去实现。

`database/sql`库提供了一些`type`。这些类型对掌握它的用法非常重要。

**`DB`** 数据库对象。 `sql.DB`类型代表了数据库。和其他语言不一样，它不是数据库连接。golang中的连接来自内部实现的连接池，连接的建立是惰性的，当你需要连接的时候，连接池会自动帮你创建。通常你不需要操作连接池。一切都有`Go`来帮你完成。

**`Results`** 结果集。数据库查询的时候，都会有结果集。`sql.Rows`类型表示查询返回多行数据的结果集。`sql.Row`则表示单行查询结果的结果集。当然，对于插入更新和删除，返回的结果集类型为`sql.Result`。

**`Statements`** 语句。`sql.Stmt`类型表示`sql`查询语句，例如`DDL`，`DML`等类似的`sql`语句。可以把当成`prepare`语句构造查询，也可以直接使用`sql.DB`的函数对其操作。

### Go的Mysql库

比较常用的Go版本的Mysql库是[go-sql-driver](https://github.com/go-sql-driver/mysql)，这都是用纯Go编写，简单高效。

对于其他语言，查询数据的时候需要创建一个连接，对于`go`而言则是需要创建一个数据库抽象对象。连接将会在查询需要的时候，由连接池创建并维护。使用`sql.Open`函数创建数据库对象。它的第一个参数是数据库驱动名，第二个参数是一个连接字串（符合`DSN`风格，可以是一个`tcp`连接，一个`unix socket`等）。

```go
import (
    "database/sql"
    "log"
    _ "github.com/go-sql-driver/mysql"
)

func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/test?parseTime=true")
    if err != nil{
        log.Fatal(err)
    }
    defer db.Close()
}
```

创建了数据库对象之后，在函数退出的时候，需要释放连接，即调用`sql.Close`方法。例子使用了`defer`语句设置释放连接。

接下来进行一些基本的数据库操作，首先我们使用`Exec`方法执行一条`sql`，创建一个数据表：

```go
func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/test?parseTime=true")
    if err != nil{
        log.Fatal(err)
    }
    defer db.Close()

    _, err = db.Exec("CREATE TABLE IF NOT EXISTS test.hello(world varchar(50))")
    if err != nil{
        log.Fatalln(err)
    }
}
```

此时可以看见，数据库生成了一个新的表。接下来再插入一些数据。

```go
func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/test?parseTime=true")

    ...

    rs, err := db.Exec("INSERT INTO test.hello(world) VALUES ('hello world')")
    if err != nil{
        log.Fatalln(err)
    }
    rowCount, err := rs.RowsAffected()
    if err != nil{
        log.Fatalln(err)
    }
    log.Printf("inserted %d rows", rowCount)
}
```

同样使用`Exec`方法即可插入数据，返回的结果集对象是是一个`sql.Result`类型，它有一个`LastInsertId`方法，返回插入数据后的`id`。当然此例的数据表并没有`id`字段，就返回一个`0`.

插入了一些数据，接下来再简单的查询一下数据：

```go
func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/test?parseTime=true")
    ... 
    rows, err := db.Query("SELECT world FROM test.hello")
    if err != nil{
        log.Fatalln(err)
    }

    for rows.Next(){
        var s string
        err = rows.Scan(&s)
        if err !=nil{
            log.Fatalln(err)
        }
        log.Printf("found row containing %q", s)
    }
    rows.Close()
}
```

我们使用了`Query`方法执行`select`查询语句，返回的是一个`sql.Rows`类型的结果集。迭代后者的`Next`方法，然后使用`Scan`方法给变量`s`赋值，以便取出结果。最后再把结果集关闭（释放连接）。

通过上面一个简单的例子，介绍了`database/sql`的基本数据查询操作。而对于开篇所说的几个结构类型尚未进行详细的介绍。下面我们再针对`database/sql`库的类型和数据库交互做更深的探究。

### sql.DB

正如上文所言，`sql.DB`是数据库的抽象，虽然通常它容易被误以为是数据库连接。它提供了一些跟数据库交互的函数，同时管理维护一个数据库连接池，帮你处理了单调而重复的管理工作，并且在多个`goroutines`也是十分安全。

`sql.DB`表示是数据库抽象，因此你有几个数据库就需要为每一个数据库创建一个`sql.DB`对象。因为它维护了一个连接池，因此不需要频繁的创建和销毁。它需要长时间保持，因此最好是设置成一个全局变量以便其他代码可以访问。

创建数据库对象需要引入标准库`database/sql`，同时还需要引入驱动`go-sql-driver/mysql`。使用`_`表示引入驱动的变量，这样做的目的是为了在你的代码中不至于和标注库的函数变量`namespace`冲突。

### 连接池

只用`sql.Open`函数创建连接池，可是此时只是初始化了连接池，并没有创建任何连接。连接创建都是惰性的，只有当你真正使用到连接的时候，连接池才会创建连接。连接池很重要，它直接影响着你的程序行为。

连接池的工作原来却相当简单。当你的函数(例如`Exec`，`Query`)调用需要访问底层数据库的时候，函数首先会向连接池请求一个连接。如果连接池有空闲的连接，则返回给函数。否则连接池将会创建一个新的连接给函数。一旦连接给了函数，连接则归属于函数。函数执行完毕后，要不把连接所属权归还给连接池，要么传递给下一个需要连接的（`Rows`）对象，最后使用完连接的对象也会把连接释放回到连接池。

请求一个连接的函数有好几种，执行完毕处理连接的方式稍有差别，大致如下：

- `db.Ping()` 调用完毕后会马上把连接返回给连接池。
- `db.Exec()` 调用完毕后会马上把连接返回给连接池，但是它返回的`Result`对象还保留这连接的引用，当后面的代码需要处理结果集的时候连接将会被重用。
- `db.Query()` 调用完毕后会将连接传递给`sql.Rows`类型，当然后者迭代完毕或者显示的调用`.Clonse()`方法后，连接将会被释放回到连接池。
- `db.QueryRow()`调用完毕后会将连接传递给`sql.Row`类型，当`.Scan()`方法调用之后把连接释放回到连接池。
- `db.Begin()` 调用完毕后将连接传递给`sql.Tx`类型对象，当`.Commit()`或`.Rollback()`方法调用后释放连接。

因为每一个连接都是惰性创建的，如何验证`sql.Open`调用之后，`sql.DB`对象可用呢？通常使用`db.Ping()`方法初始化：

```go
db, err := sql.Open("driverName", "dataSourceName")
if err != nil{
    log.Fatalln(err)
}

defer db.Close()

err = db.Ping()
if err != nil{
   log.Fatalln(err)
}
```

调用了`Ping`之后，连接池一定会初始化一个数据库连接。当然，实际上对于失败的处理，应该定义一个符合自己需要的方式，现在为了演示，简单的使用`log.Fatalln(err)`表示了。

#### 连接失败

关于连接池另外一个知识点就是你不必检查或者尝试处理连接失败的情况。当你进行数据库操作的时候，如果连接失败了，`database/sql`会帮你处理。实际上，当从连接池取出的连接断开的时候，`database/sql`会自动尝试重连10次。仍然无法重连的情况下会自动从连接池再获取一个或者新建另外一个。好比去买鸡蛋，售货员会从箱子里掏出鸡蛋，如果发现是坏蛋则连续掏10次，仍然找不到合适的就会换一个箱子招，或者从别的库房再拿一个给你。

#### 连接池配置

无论哪一个版本的`go`都不会提供很多控制连接池的接口。知道`1.2`版本以后才有一些简单的配置。可是`1.2`版本的连接池有一个`bug`，请升级更高的版本。

配置连接池有两个的方法：

- `db.SetMaxOpenConns(n int)` 设置打开数据库的最大连接数。包含正在使用的连接和连接池的连接。如果你的函数调用需要申请一个连接，并且连接池已经没有了连接或者连接数达到了最大连接数。此时的函数调用将会被`block`，直到有可用的连接才会返回。设置这个值可以避免并发太高导致连接`mysql`出现`too many connections`的错误。该函数的默认设置是0，表示无限制。
- `db.SetMaxIdleConns(n int)` 设置连接池中的保持连接的最大连接数。默认也是`0`，表示连接池不会保持释放会连接池中的连接的连接状态：即当连接释放回到连接池的时候，连接将会被关闭。这会导致连接再连接池中频繁的关闭和创建。

对于连接池的使用依赖于你是如何配置连接池，如果使用不当会导致下面问题：

1. 大量的连接空闲，导致额外的工作和延迟。
2. 连接数据库的连接过多导致错误。
3. 连接阻塞。
4. 连接池有超过十个或者更多的死连接，限制就是`10`次重连。

大多数时候，如何使用`sql.DB`对连接的影响大过连接池配置的影响。这些具体问题我们会再使用`sql.DB`的时候逐一介绍。



## CURD基础操作

### 读取数据

`database/sql`提供了`Query`和`QueryRow`方法进行查询数据库。对于`Query`方法的原理，正如前文介绍的主要分为三步：

1. 从连接池中请求一个连接
2. 执行查询的`sql`语句
3. 将数据库连接的所属权传递给`Result`结果集

`Query`返回的结果集是`sql.Rows`类型。它有一个`Next`方法，可以迭代数据库的游标，进而获取每一行的数据，大概使用范式如下：

```go
rows, err := db.Query("SELECT world FROM test.hello")
if err != nil{
    log.Fatalln(err)
}

for rows.Next(){
    var s string
    err = rows.Scan(&s)
    if err !=nil{
        log.Fatalln(err)
    }
    log.Printf("found row containing %q", s)
}
rows.Close()
```

`rows.Next`方法设计用来迭代。当它迭代到最后一行数据之后，会触发一个`io.EOF`的信号，即引发一个错误，同时`go`会自动调用`rows.Close`方法释放连接，然后返回`false`。此时循环将会结束退出。

通常你会正常迭代完数据然后退出循环。可是如果并没有正常的循环而因其他错误导致退出了循环。此时`rows.Next`处理结果集的过程并没有完成，归属于`rows`的连接不会被释放回到连接池。因此十分有必要正确的处理`rows.Close`事件。如果没有关闭`rows`连接，将导致大量的连接并且不会被其他函数重用，就像溢出了一样。最终将导致数据库无法使用。

那么如何阻止这样的行为呢？上述代码已经展示，无论循环是否完成或者因为其他原因退出，都显示的调用`rows.Close`方法，确保连接释放。又或者使用`defer`指令在函数退出的时候释放连接，即使连接已经释放了，`rows.Close`仍然可以调用多次，是无害的。

使用`defer`的时候需要注意，如果一个函数执行很长的逻辑，例如`main`函数，那么`rows`的连接释放就会也很长，好的实践方案是尽可能的越早释放连接。

`rows.Next`循环迭代的时候，因为触发了`io.EOF`而退出循环。为了检查是否是迭代正常退出还是异常退出，需要检查`rows.Err`。例如上面的代码应该改成：

```go
rows, err := db.Query("SELECT world FROM test.hello")
if err != nil{
    log.Fatalln(err)
}
defer rows.Close()

for rows.Next(){
    var s string
    err = rows.Scan(&s)
    if err !=nil{
        log.Fatalln(err)
    }
    log.Printf("found row containing %q", s)
}
rows.Close()
if err = rows.Err(); err != nil {
    log.Fatal(err)
}
```

### 读取单条记录

`Query`方法是读取多行结果集，实际开发中，很多查询只需要单条记录，不需要再通过`Next`迭代。`golang`提供了`QueryRow`方法用于查询单条记录的结果集。

```go
var s string
err = db.QueryRow("SELECT world FROM test.hello LIMIT 1").Scan(&s)
if err != nil{
    if err == sql.ErrNoRows{
        log.Println("There is not row")
    }else {
        log.Fatalln(err)
    }
}
log.Println("found a row", s)
```

`QueryRow`方法的使用很简单，它要么返回`sql.Row`类型，要么返回一个`error`，如果是发送了错误，则会延迟到`Scan`调用结束后返回，如果没有错误，则`Scan`正常执行。只有当查询的结果为空的时候，会触发一个`sql.ErrNoRows`错误。你可以选择先检查错误再调用`Scan`方法，或者先调用`Scan`再检查错误。

### rows.Scan原理

结果集方法`Scan`可以把数据库取出的字段值赋值给指定的数据结构。它的参数是一个空接口切片`func (rs *Rows) Scan(dest ...interface{}) error`，这就意味着可以传入任何值。通常把需要赋值的目标变量的指针当成参数传入，它能将数据库取出的值赋值到指针值对象上。

```go
var var1, var2 string
err = row.Scan(&var1, &var2)
```

在一些特殊案例中，如果你不想把值赋值给指定的目标变量，那么需要使用`*sql.RawBytes`类型。如何使用`*sql.RawBytes`需要参考更细的官方文档。大多数情况下我们不必这么做。但是还是需要注意在`db.QueryRow().Scan()`中不能使用`*sql.RawBytes`。

`Scan`还会帮我们自动推断除数据字段匹配目标变量。比如有个数据库字段的类型是`VARCHAR`，而他的值是一个数字串，例如`"1"`。如果我们定义目标变量是`string`，则`scan`赋值后目标变量是数字`string`。如果声明的目标变量是一个数字类型，那么`scan`会自动调用`strconv.ParseInt()`或者`strconv.ParseInt()`方法将字段转换成和声明的目标变量一致的类型。当然如果有些字段无法转换成功，则会返回错误。因此在调用`scan`后都需要检查错误。

```go
var world int
err = stmt.QueryRow(1).Scan(&world)
```

此时`scan`会把字段转变成数字整型的赋值给`world`变量

```go
var world string
err = stmt.QueryRow(1).Scan(&world)
```

此时`scan`取出的字段就是字串。同样的如果字段是`int`类型，声明的变量是`string`类型，`scan`也会自动将数字转换成字串赋值给目标变量。

### 空值处理

数据库有一个特殊的类型，`NULL`空值。可是`NULL`不能通过`scan`直接跟普遍变量赋值，甚至也不能将null赋值给`nil`。对于`null`必须指定特殊的类型，这些类型定义在`database/sql`库中。例如`sql.NullFloat64`。如果在标准库中找不到匹配的类型，可以尝试在驱动中寻找。下面是一个简单的例子：

```go
var (
   s1 string
    s2 sql.NullString i1 int
    f1 float64
    f2 float64
)
// 假设数据库的记录为 ["hello", NULL, 12345, "12345.6789", "not-a-float"]
err = rows.Scan(&s1, &s2, &i1, &f1, &f2) if err != nil {
log.Fatal(err) }
```

因为最后一个`f2`字段的值不是`float`，这会英法一个错误。

```go
sql: Scan error on column index 4: converting string "not-a- oat" to a  oat64: strconv.ParseFloat: parsing "not-a- oat": invalid syntax
```

如果忽略`err`，强行读取目标变量，可以看到最后一个值转换错误会处理，而不是抛出异常：

```go
err = rows.Scan(&s1, &s2, &i1, &f1, &f2)
log.Printf("%q %#v %d %f %f", s1, s2, i1, f1, f2)

// 输出
 "hello" sql.NullString{String:"", Valid:false} 12345 12345.678900
0.000000
```

可以看到，除了最后一个转换失败变成了零值之外，其他都正常的取出了值，尤其是`null`匹配了`NullString`类型的目标变量。

对于`null`的操作，通常仍然需要验证：

```go
var world sql.NullString
err := db.QueryRow("SELECT world FROM hello WHERE id = ?", id).Scan(&world)
...
if world.Valid {
      wrold.String 
} else {
    // 数据库的value是不是null的时候，输出 world的字符串值， 空字符串   
    world.String
}
```

对应的，如果`world`字段是一个`int`，那么声明的目标变量类似是`sql.NullInt64`，读取其值的方法为`world.Int64`。

但是有时候我们并不关心值是不是`Null`,我们只需要吧他当一个空字符串来对待就行。这时候我们可以使用`[]byte（null byte[]`可以转化为空`string`）

```go
var world []byte
err := db.QueryRow("SELECT world FROM hello WHERE id = ?", id).Scan(&world)
...
log.Println(string(real_name)) // 有值则取出字串值，null则转换成 空字串。
```

### 自动匹配字段

在执行查询的时候，我们定义了目标变量，同时查询的时候也写明了字段，如果不指名字段，或者字段的顺序和查询的不一样，都有可能出错。因此如果能够自动匹配查询的字段值，将会十分节省代码，同时也易于维护。

`go`提供了`Columns`方法用获取字段名，与大多数函数一样，读取失败将会返回一个`err`，因此需要检查错误。

```go
cols, err := rows.Columns()
if err != nil{
   log.Fatalln(er)
}
```

对于不定字段查询，我们可以定义一个`map`的`key`和`value`用来表示数据库一条记录的`row`的值。通过`rows.Columns`得到的`col`作为`map`的`key`值。下面是一个例子

```go
func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/test?parseTime=true")
    if err != nil{
        log.Fatal(err)
    }
    defer db.Close()

    rows, err := db.Query("SELECT * FROM user WHERE gid = 1")
    if err != nil{
        log.Fatalln(err)
    }
    defer rows.Close()


    cols, err := rows.Columns()
    if err != nil{
        log.Fatalln(err)
    }
    fmt.Println(cols)
    vals := make([][]byte, len(cols))
    scans := make([]interface{}, len(cols))

    for i := range vals{
        scans[i] = &vals[i]
    }

    var results []map[string]string

    for rows.Next(){
        err = rows.Scan(scans...)
        if err != nil{
            log.Fatalln(err)
        }

        row := make(map[string]string)
        for k, v := range vals{
            key := cols[k]
            row[key] = string(v)
        }
        results = append(results, row)
    }

    for k, v :=range results{
        fmt.Println(k, v)
    }
}
```

数据表`user`有三个字段，`id（int）`，`gid（int）`，`real_name（varchar）`。我们使用`*`取出所有的字段。使用`rows.Columns()`获取字段名，是一个`string`的数组。然后创建一个切片`vals`，用来存放所取出来的数据结果，类似是`byte`的切片。接下来还需要定义一个切片，这个切片用来`scan`，将数据库的值复制到给它。

完成这一步之后，`vals`则得到了`scan`复制给他的值，因为是`byte`的切片，因此在循环一次，将其转换成`string`即可。

转换后的`row`即我们取出的数据行值，最后组装到`result`切片中。

运行结果如下

```go
[id gid real_name]
0 map[id:4 gid:1 real_name:瑟兰督依]
1 map[real_name:来格拉斯 id:5 gid:1]
2 map[id:15 gid:1 real_name:]
```

有一条记录的`real_name` 值为空字串，因为其数据库存储的是NULL。

### Exec

前面介绍了很多关于查询方面的内容，查询是读方便的内容，对于写，即插入更新和删除。这类操作与`query`不太一样，写的操作只关系是否写成功了。`database/sql`提供了`Exec`方法用于执行写的操作。

我们也见识到了，`Eexc`返回一个`sql.Result`类型，它有两个方法`LastInsertId`和`RowsAffected`。`LastInsertId`返回是一个数据库自增的`id`，这是一个`int64`类型的值。

`Exec`执行完毕之后，连接会立即释放回到连接池中，因此不需要像`query`那样再手动调用`row`的`close`方法。

关于`LastInsertId`和`RowsAffected`方法执行错误的返回，这跟底层的数据库是有关系的。因此更多的细节可以参考驱动的文档。

### 总结

目前，我们大致了解了数据库的CURD操作。对于读的操作，需要定义目标变量才能`scan`数据记录。`scan`会智能的帮我们转换一些数据，取决于定义的目标变量类型。对于`null`的处理，可以使用`database/sql`或驱动提供的类型声明，也可以使用`[]byte`将其转换成空字串。除了读数据之外，对于写的操作，`database/sql`也提供了`Exec`方法，并且对于`sql.Result`提供了`LastInsertId`和`RowsAffected`方法用于获取写后的结果。

在实际应用中，与数据库交互，往往写的`sql`语句还带有参数，这类`sql`可以称之为`prepare`语句。`prepare`语句有很多好处，可以防止`sql`注入，可以批量执行等。但是`prepare`的连接管理有其自己的机制，也有其使用上的陷进，关于`prepare`的使用，我们将会在以后讨论。

## Prepared剖析

### mysql的sql处理过程

#### 即时`SQL`
一条 SQL 在 DB 接收到最终执行完毕返回，大致的过程如下：

  1. 词法和语义解析；

  2. 优化`SQL`语句，制定执行计划；

  3. 执行并返回结果；
      如上，一条`SQL`直接是走流程处理，一次编译，单次运行，此类普通语句被称作`Immediate Statements` （即时`SQL`）。

#### 预处理`SQL`

但是，绝大多数情况下，某需求某一条`SQL`语句可能会被反复调用执行，或者每次执行的时候只有个别的值不同（比如`select`的`where`子句值不同，`update`的`set`子句值不同，`insert` 的`values`值不同）。如果每次都需要经过上面的词法语义解析、语句优化、制定执行计划等，则效率就明显不行了。
所谓预编译语句就是将此类`SQL`语句中的值用占位符替代，可以视为将`SQL`语句模板化或者说参数化，一般称这类语句叫`Prepared Statements`。预编译语句的优势在于归纳为：一次编译、多次运行，省去了解析优化等过程；此外预编译语句能防止`SQL`注入。

### 自定义`prepare`查询

从`query`查询可以看到，对于占位符的`prepare`语句，`go`内部通过的`dc.ci.Prepare(query)`会自动创建一个 `stmt`对象。其实我们也可以自定义`stmt`语句，使用方式如下：

```go
stmt, err := db.Prepare("SELECT * FROM user WHERE gid = ?")
if err != nil {
    log.Fatalln(err)
}
defer stmt.Close()

rows, err :=  stmt.Query(1)
if err != nil{
    log.Fatalln(err)
}
```

即通过`Prepare`方法创建一个`stmt`对象，然后执行`stmt`对象的`Query（Exec）`方法得到`sql.Rows`结果集。最后关闭`stmt.Close`。这个过程就和之前所说的`prepare`三步骤匹配了。

创建`stmt`的`preprea`方式是`golang`的一个设计，其目的是`Prepare once, execute many times`。为了批量执行`sql`语句。但是通常会造成所谓的三次网络请求（`three network round-trips`）。即`preparing`、 `executing`和`closing`三次请求。

对于大多数数据库，`prepread`的过程都是，先发送一个带占位符的`sql`语句到服务器，服务器返回一个`statement id`，然后再把这个`id`和参数发送给服务器执行，最后再发送关闭`statement`命令。

`golang`的实现了连接池，处理`prepare`方式也需要特别注意。调用`Prepare`方法返回`stmt`的时候，`golang`会在某个空闲的连接上进行`prepare`语句，然后就把连接释放回到连接池，可是`golang`会记住这个连接，当需要执行参数的时候，就再次找到之前记住的连接进行执行，等到`stmt.Close`调用的时候，再释放该连接。

在执行参数的时候，如果记住的连接正处于忙碌阶段，此时`golang`将会从新选一个新的空闲连接进行`prepare（re-prepare）`。当然，即使是重新`reprepare`，同样也会遇到刚才的问题。那么将会一而再再而三的进行`reprepare`。直到找到空闲连接进行查询的时候。

这种情况将会导致`leak`连接的情况，尤其是再高并发的情景。将会导致大量的`prepare`过程。因此使用`stmt`的情况需要仔细考虑应用场景，通常在应用程序中。多次执行同一个`sql`语句的情况并不多，因此减少`prepare`语句的使用。

之前有一个疑问，是不是所有`sql`语句都不能带占位符，因为这是`prepare`语句。只要看了一遍`database/sql`和驱动的源码才恍然大悟，对于`query(prepare, args)`的方式，正如我们前面所分析的，`database/sql`会使用`ds.si.Query(dargs)`创建`stmt`的，然后就立即执行`prepare`和参数，最后关闭`stmt`。整个过程都是同一个连接上完成，因此不存在`reprepare`的情况。当然也无法使用所谓的创建一次，执行多次的目。

对于`prepare`的使用方式，基于其好处和缺点，我们将会再后面的最佳实践再讨论。目前需要注意的大致就是：

1. 单次查询不需要使用`prepared`，每次使用`stmt`语句都是三次网络请求次数，`prepared execute close`
2. 不要循环中创建`prepare`语句
3. 注意关闭`stmt`

尽管会有`reprepare`过程，这些操作依然是`database/sql`帮我们所做的，与连接`retry 10`次一样，开发者无需担心。

对于`Qeruy`操作如此，同理`Exec`操作也一样。

### 总结

目前我们学习`database/sql`提供两类查询操作，`Query`和`Exec`方法。他们都可以使用`plaintext`和`preprea`方式查询。对于后者，可以有效的避免数据库注入。而`prepare`方式又可以有显示的声明`stmt`对象，也有隐藏的方式。显示的创建`stmt`会有`3`次网络请求，创建->执行->关闭，再批量操作可以考虑这种做法，另外一种方式创建`prepare`后就执行，因此不会因为`reprepare`导致高并发下的leak连接问题。

具体使用那种方式，还得基于应用场景，安全过滤和连接管理等考虑。至此，关于查询和执行操作已经介绍了很多。关系型数据库的另外一个特性就是关系和事务处理。下一节，我们将会讨论`database/sql`的数据库事务功能。



## 事物

### Transaction 事务

事务处理是数据的重要特性。尤其是对于一些支付系统，事务保证性对业务逻辑会有重要影响。`golang`的`mysql`驱动也封装好了事务相关的操作。我们已经学习了`db`的`Query`和`Exec`方法处理查询和修改数据库。

### tx对象

一般查询使用的是db对象的方法，事务则是使用另外一个对象。`sql.Tx`对象。使用`db`的`Begin`方法可以创建`tx`对象。`tx`对象也有数据库交互的`Query`,`Exec`和`Prepare`方法。用法和`db`的相关用法类似。查询或修改的操作完毕之后，需要调用`tx`对象的`Commit`提交或者`Rollback`方法回滚。

一旦创建了`tx`对象，事务处理都依赖与`tx`对象，这个对象会从连接池中取出一个空闲的连接，接下来的`sql`执行都基于这个连接，直到`commit`或者`rollback`调用之后，才会把连接释放到连接池。

在事务处理的时候，不能使用`db`的查询方法，虽然后者可以获取数据，可是这不属于同一个事务处理，将不会接受`commit`和`rollback`的改变，一个简单的事务例子如下：

```go
tx, err := db.Begin()
tx.Exec(query1)
tx.Exec(query2)
tx.commit()
```

在`tx`中使用`db`是错误的：
```go
tx, err := db.Begin()
db.Exec(query1)
tx.Exec(query2)
tx.commit()
```
上述代码在调用`db`的`Eexc`方法的时候，`tx`会绑定连接到事务中，`db`则是额外的一个连接，两者不是同一个事务。需要注意，`Begin`和`Commit`方法，与`sql`语句中的`BEGIN`或`COMMIT`语句没有关系。

### 事务与连接
创建`Tx`对象的时候，会从连接池中取出连接，然后调用相关的`Exec`方法的时候，连接仍然会绑定在改事务处理中。在实际的事务处理中，`go`可能创建不同的连接，但是那些其他连接都不属于该事务。例如上面例子中`db`创建的连接和`tx`的连接就不是一回事。

事务的连接生命周期从`Beigin`函数调用起，直到`Commit`和`Rollback`函数的调用结束。事务也提供了`prepare`语句的使用方式，但是需要使用`Tx.Stmt`方法创建。`prepare`设计的初衷是多次执行，对于事务，有可能需要多次执行同一个`sql`。然而无论是正常的`prepare`和事务处理，`prepare`对于连接的管理都有点小复杂。因此私以为尽量避免在事务中使用`prepare`方式。例如下面例子就容易导致错误：

```go
tx, _ := db.Begin()
defer tx.Rollback()
stmt, _ tx.Prepare("INSERT ...")
defer stmt.Close()
tx.Commit()
```

因为`stmt.Close`使用`defer`语句，即函数退出的时候再清理`stmt`，可是实际执行过程的时候，`tx.Commit`就已经释放了连接。当函数退出的时候，再执行`stmt.Close`的时候，连接可能有被使用了。

### 事务并发

对于`sql.Tx`对象，因为事务过程只有一个连接，事务内的操作都是顺序执行的，在开始下一个数据库交互之前，必须先完成上一个数据库交互。例如下面的例子：

```go
rows, _ := db.Query("SELECT id FROM user") 
for rows.Next() {
    var mid, did int
    rows.Scan(&mid)
    db.QueryRow("SELECT id FROM detail_user WHERE master = ?", mid).Scan(&did)
    
}
```

调用了`Query`方法之后，在`Next`方法中取结果的时候，`rows`是维护了一个连接，再次调用`QueryRow`的时候，`db`会再从连接池取出一个新的连接。`rows`和`db`的连接两者可以并存，并且相互不影响。

可是，这样逻辑在事务处理中将会失效：

```go
rows, _ := tx.Query("SELECT id FROM user")
for rows.Next() {
   var mid, did int
   rows.Scan(&mid)
   tx.QueryRow("SELECT id FROM detail_user WHERE master = ?", mid).Scan(&did)
}
```

`tx`执行了`Query`方法后，连接转移到`rows`上，在`Next`方法中，`tx.QueryRow`将尝试获取该连接进行数据库操作。因为还没有调用`rows.Close`，因此底层的连接属于`busy`状态，`tx`是无法再进行查询的。上面的例子看起来有点傻，毕竟涉及这样的操作，使用`query`的`join`语句就能规避这个问题。例子只是为了说明`tx`的使用问题。

### 实践

前面对事务解释了一堆，说了那么多，其实还不如share的code。下面就事务的使用做简单的介绍。因为事务是单个连接，因此任何事务处理过程的出现了异常，都需要使用`rollback`，一方面是为了保证数据完整一致性，另一方面是释放事务绑定的连接。

```go
func doSomething(){
    panic("A Panic Running Error")
}

func clearTransaction(tx *sql.Tx){
    err := tx.Rollback()
    if err != sql.ErrTxDone && err != nil{
        log.Fatalln(err)
    }
}

func main() {
    db, err := sql.Open("mysql", "root:@tcp(127.0.0.1:3306)/test?parseTime=true")
    if err != nil {
        log.Fatalln(err)
    }

    defer db.Close()

    tx, err := db.Begin()
    if err != nil {
        log.Fatalln(err)
    }
    defer clearTransaction(tx)

    rs, err := tx.Exec("UPDATE user SET gold=50 WHERE real_name='vanyarpy'")
    if err != nil {
        log.Fatalln(err)
    }
    rowAffected, err := rs.RowsAffected()
    if err != nil {
        log.Fatalln(err)
    }
    fmt.Println(rowAffected)

    rs, err = tx.Exec("UPDATE user SET gold=150 WHERE real_name='noldorpy'")
    if err != nil {
        log.Fatalln(err)
    }
    rowAffected, err = rs.RowsAffected()
    if err != nil {
        log.Fatalln(err)
    }
    fmt.Println(rowAffected)

    doSomething()

    if err := tx.Commit(); err != nil {
        // tx.Rollback() 此时处理错误，会忽略doSomthing的异常
        log.Fatalln(err)
    }

}
```

我们定义了一个`clearTransaction(tx)`函数，该函数会执行`rollback`操作。因为我们事务处理过程中，任何一个错误都会导致`main`函数退出，因此在main函数退出执行`defer`的`rollback`操作，回滚事务和释放连接。

如果不添加`defer`，只在最后`Commi`t后`check`错误`err`后再`rollback`，那么当`doSomething`发生异常的时候，函数就退出了，此时还没有执行到`tx.Commit`。这样就导致事务的连接没有关闭，事务也没有回滚。

### 总结

`database/sql`提供了事务处理的功能。通过Tx对象实现。`db.Begin`会创建`tx`对象，后者的`Exec`和`Query`执行事务的数据库操作，最后在`tx`的`Commit`和`Rollback`中完成数据库事务的提交和回滚，同时释放连接。

`tx`事务环境中，只有一个数据库连接，事务内的`Eexc`都是依次执行的，事务中也可以使用`db`进行查询，但是`db`查询的过程会新建连接，这个连接的操作不属于该事务。



## 引用

* [Golang Mysql笔记（一）--- 连接与连接池](https://www.jianshu.com/p/340eb943be2e)
* [Golang Mysql笔记（二）--- CURD基础](https://www.jianshu.com/p/50c9fbf4046c)
* [Golang Mysql笔记（三）--- Prepared剖析](https://www.jianshu.com/p/ee0d2e7bef54)
* [Golang Mysql笔记（四）--- 事务](https://www.jianshu.com/p/bc8120bec94e)

* [go-sql-driver](https://github.com/go-sql-driver/mysql)