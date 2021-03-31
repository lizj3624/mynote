# lua程序设计(第4版)
## lua词法规则以及变量
* lua标识符(变量)可以使由任意字母、数字和下划线构成的字符串，但是不能以数字开头
* 应该避免使用以一个下划线开头并跟着一个或者多个大写字母(`_VERSION`)的标识符,这种在lua中有特殊用途
* lua变量是有大小写区分的
* lua使用`--`进行单行注释,使用`--[[  --]]`进行代码块注释
```lua
--[[
print(10)
--]]
```
* 局部/全局变量
```lua
j = 10         --全局变量
local i = 5    --局部变量,只适用于具体的代码块中
```
可以通过`do-end`严格控制代码块的作用域

## lua的类型与值
* lua有8种基础类型：`nil`, `boolean`, `number`, `string`, `userdata`, `function`, `thread`, `table`

## lua逻辑操作符
lua的逻辑操作符有`and`、`or`、`not`, 与条件语句一样，所有的逻辑操作符将`false`和`nil`视为假，而将其他的任何东西视为真。
* `and`：如果它的第一操作数为假，将将返回第一个操作数；不然返回第二操作数。
* `or`：如果它的第一操作数为真，就返回第一个操作数；不然返回第二个操作数。
* `and`和`or`都使用"短路求值"，就是只有在需要时才去评估第二个操作数。
```lua
print(4 and 5)
5

print(nil and 13)
nil

print(false and 13)
false

print(4 or 5)
4

print(false or 5)
5

print(nil or 5)
5
```

## lua函数
1. lua中的函数

```lua
# 求和函数
function add (a)
    local sum = 0
    for i = 1, #a do
        sum = sum + a[i]
    end
    
    return sum
end
```

调用函数时使用的参数个数可以与定义函数时使用的参数个数不一致，Lua会通过抛弃多余参数和将不足的参数设置nil的方式来调整参数的个数。

2. 返回多值

    Lua允许一个函数返回多个结果。返回多值时只要`return`后列出所有要返回的值，`return`不需要加括号。

    Lua语言根据函数的被调用情况调整返回值的数量。 

    * 当函数被作为一条单独语句调用时， 其所有返回值都会被丢弃
    * 当函数被作为表达式(例如，加法的操作数)调用时，将只保留 函数的第一个返回值。 
    * 只有当函数调用是一系列表达式中的最后一个表达式(或是唯一一个表达式)时，其所有的返 回值才能被获取到 。 这里所谓的“一系列表达式”在 Lua 中表现为 4种情况 : 多重赋值、函数调用时传入的实参列表、表构造器和 return语句 。

    ```lua
    function foo0 () end
    function foo1 () return "a" end
    function foo2 () return "a", "b" end
    
    x, y = foo2()   -- x = "a" y = "b"
    x = foo2()      -- x = "a", 被丢弃
    x, y = foo1()   -- x = "a", b = nil
    ```

3. 可变长参数函数

    ```lua
    function add (...)
        local s = 0
        for _, v in ipairs(...) do
            s = s + v
        end
        
        return s
    end
    ```

    参数列表中的三个点`...`表示该函数的参数是可变长的。

4. 函数 table.unpack

5. ```lua
print(table.unpack{10,20,30})       -- 10 20 30
    a,b = table.unpack{10,2日，30}      -- a=10, b=20, 30被丢弃
  ```
  
    顾名思义，函数`table.unpack`与函数`table.pack`的功能相反，`pack`把参数列表转换成`Lua`语言中一个真实的列表(一个表)， 而`unpack`则把`Lua`语言中的真实的列表(一个表) 转换成一组返回值，进而可以作为另一个函数的参数被使用。

6. 正确的尾调用

    尾调用 (tail call) 是被当作函数调用使用的跳转。 当一个函数的最后一个动作是调用另一个函数而没有再进行其他工作时，就形成了尾调用。 例如，下列代码中对函数的调用就是尾调用 :

    ```lua
    function f (x) x =x + 1; return g(x) end
    ```

   当`g`返回时，程序的执行路径会直接返回到调用`f`的位置。在一些语言的实现中，例如Lua语言解释器，就利用了这个特点，使得在进行尾调用时不使 用任何额外的枝空间。我们就将这种实现称为尾调用消除( *ta**il**-ca**ll elimination* )。在 Lua语言中，只有形如`return func(args) `的调用才是尾调用 。

### 闭合函数(closure)

若将一个函数写在另一个函数之内，那么这个位于内部的函数便可以访问外部函数中的局部变量，这项特征称之为“词法域”, 词法域和“第一类的”函数时两项极其有用的概念。
```lua
function sortbygrade (names, grades)
    table.sort(names, function(n1, n2)
            return grades[n1] > grades[n2]
        end)
end
-- grades是外函数sortbygrade的形参，但是内部函数table.sort中，grades既不是全局变量也不是局部变量, 但是table.sort是可以访问这个变量grades

function newCounter ()
    local i = 0
    return function ()
               i = i + 1
               return i
           end
end

内部匿名函数可以访问局部变量`i`
```

## 输入输出

1. 简单I/O模型

    ```lua
    io.write(a, b, c)
    io.writer(string.format("sin(3) = %.4f\n", math.sin(3)))
    
    t = io.read("a")
    t = string.gsub(t, "bad", "good")
    io.write(t)
    ```

2. 完整的I/0模型

```lua
local f = assert(io.open(filenname, "r"))

local t = f:read("a")

f:close()
```



3、系统调用

```lua
print(os.getenv("HOME"))

function CreateDir(dirname)
    os.execute("mkdir "...dirname)
end

local f = io.popen("dir /B", "r")
local dir = {}
for entry in f:lines() do
    dir[#dir + 1] = entry
end
```



## 变量作用域以及结构控制

1. Lua语言中的变量在默认情况下是全局变量，所有的局部变量在使用前必须声明。 与全局变量不同，局部变量的生效范围仅限于声明它的代码块。一个代码块 (block)是一个控制结构的主体，或是一个函数的主体，或是一个代码段(即变量被声明时所在的文件或字符串):

```lua
x =10                       -- 全局的
local i = 1                 --对于代码段来说是局部的

while i <= x do 
    local x = i * 2         -- 对于循环体来说是局部的
    print(x) 
    i = i + 1
end

if i > 20 then 
    local x                 --对于”then”来说是局部的
    x = 20
    print(x + 2)            --(如采测试成功会输出 22 )
else
    print( x)               --10 (全局的)，与"then"的x不是一个
end

print(x)                    --10 (全局的)

-- 使用do-end语句块控制局部变量
local x1, x2 
do
    local a2 = 2*a
    local d = (b^2 - 4*a*c)^(1/2)
    x1 = (-b + d)/a2
    x2 = (-b - d)/a2
end                          --’a2’和’d’的范围在此结束 
print(x1, x2)                --’x1’和’x2’仍在范围内
```

2. 控制结构

   Lua将所有不是`false`和`nil`的值当作真，特别注意是：Lua语言会将`0`或者空字符也当作真

   a、if-then-else

   ```lua
   if a < 0 then
    a = 0
   end
   ```

if a < b then
    return a
else 
    return b
end

if op == "+" then
    r = a + b
elseif op == "-" then
    r = a - b
elseif op == "*" then
    r = a * b
elseif op == "/" then
    r = a / b
else
    error("invaild op")
end
   ```

Lua语言不支持switch

   b、while

   ```lua
local i = 1
while a[i] do
    print(a[i])
    i = i + 1
end
   ```

   c、repeat

```lua
一输出第一个非空的行
local line 
repeat
    line =io.read() 
until line ~= ""
print(line)
```

   d、for

数值型，

```lua
for var = exp1, exp2, exp3 do
    something
end
-- 第三个表达式exp3可选，代表是步长，不存在时默认是1
-- 在循环之前，三个表达式都会运行一次；控制变量var是自动声明的局部变量，在范围仅在循环体内

for i = 1, 10 do
    print(i)
end
```

泛型for遍历迭代函数的所有返回值，可以使用多个变量，这些变量在每次循环时都会更新，当第一个变量变为nil时，循环终止。

```lua
for v, k in ipairs()
end
```

e、break，return，goto，Lua里没有continue

## 闭包

"第一类值"意味Lua语言中的函数与其他常见类型的值具有相同权限；函数可以保持变量中，表中，当作函数参数，其他函数的返回值。

"词法定界"意味着Lua语言中的函数可以访问包含自身的外部函数中的变量。

### 函数是第一类值

在lua中函数是一种"第一类值", 它们具有特定的词法域，函数跟其他传统的类型的值具有相同的权利:

* 函数可以存储在变量中或者table中
* 函数可以作为实参传递给其他函数
* 函数可以作为其他函数的返回值
* 具有特定的词法域，一个函数中可以嵌套在另一个函数中，内部函数可以访问外部函数的变量

```lua
a = {p = print}
a.p("Hello World")       --Hello World
print = math.sin         --'print'现在引用math.sin的正玄函数
a.p(print(1))            --math.sin
sin = a.p                --sin = print
sin(10, 20)

-- 将匿名函数赋值给局部变量
local access_ok = function (f, mode)
end

-- 函数语法糖
foo = function (x) return 2*x end
```

Lua语言中所有的函数都是匿名函数

像函数sort这样以另一个函数为参数的函数，我们称之为高阶函数。

```lua
table.sort(network, function(a,b) return (a.name > b,.name) end)
```

### 非全局函数

1. 局部函数，当把函数存储在局部变量中时，这个函数就称为了局部函数，一个被限定在指定作用域中使用的函数。

```lua
-- lua定义局部函数
local function f (params)
    body
end

-- 局部函数递归时，要提前声明局部变量

local foo; foo = function (params) body end
```

### 词法定界

当编写一个被其他函数B包含的函数A时，被包含的函数A可以访问包含其的函数B的所有局部变量，这种特性称为词法定界。

```lua
names = {"Peter", "Paul", "Mary"}
grages = {Mary = 10, Paul = 7, Peter = 8}

function sortbygrage(name, grades)
    table.sort(names, function(n1, n2) return grades[n1] > grades[n2] end)
end
```

`sort`中的匿名函数可以访问`grades`，而`grades`是包含匿名函数外的函数`sortbygrade`的形参

## 位与字节

1. 按位与 `&`
2. 按位或 `|`
3. 按位异或 `~`
4. 逻辑右移 `>>`
5. 逻辑左移 `<<`
6. 按位取反 `~`

## Lua API用法

### 字符串API

#### string.format

虽然lua中字符串拼接`string.format`相对于`..`消耗较大，但有时为了代码的可读性，项目中还是经常用到`string.format`。至于这两个用法的性能看源码也很容易看出来，这里就简单说一下，前者其实调用C函数`str_format`来实现拼接的，而后者只是一个操作符，通过`memcpy`来拼接，并且多个`..`的操作其实也只执行了一次`concat`。

常用转义符：

`%c` - 接受一个数字, 并将其转化为ASCII码表中对应的字符
`%d, %i` - 接受一个数字并将其转化为有符号的整数格式
`%o` - 接受一个数字并将其转化为八进制数格式
`%u` - 接受一个数字并将其转化为无符号整数格式
`%x` - 接受一个数字并将其转化为十六进制数格式, 使用小写字母
`%X` - 接受一个数字并将其转化为十六进制数格式, 使用大写字母
`%e` - 接受一个数字并将其转化为科学记数法格式, 使用小写字母e
`%E` - 接受一个数字并将其转化为科学记数法格式, 使用大写字母E
`%f` - 接受一个数字并将其转化为浮点数格式
`%g(%G)` - 接受一个数字并将其转化为`%e`(`%E`, 对应`%G`)及`%f`中较短的一种格式
`%q` - 接受一个字符串并将其转化为可安全被Lua编译器读入的格式
`%s` - 接受一个字符串并按照给定的参数格式化该字符串

实用扩展：

对于`string.format`的使用，转义符的使用也是有部分技巧。

1）`string.format`中怎么匹配带`%`的的字符串和转义符的使用

```lua
string.format("%d%%", 100)   --输出:  100%

string.format("\"%s\"", "Hello World")   --输出:   "Hello World"
```



2)常用的格式控制符

可以在`%`号后添加参数. 参数将以如下的顺序读入:

(1) 符号: 一个`+`号表示其后的数字转义符将让正数显示正号. 负数不变.

(2) 占位符: 一个`0`, 在后面指定了字串宽度时占位用. 默认占位符是空格.

(3) 对齐标识: 在指定了字串宽度时, 默认为右对齐, 增加-号可以改为左对齐.（用于一些自动空格地方）

(4) 宽度数值 `.`小数位数/字串裁切: 在宽度数值后增加的小数部分n, 若后接f则设定该浮点数的小数只保留n位, 若后接s则设定该字符串只显示前n位.

```lua
string.format("%05d", 2015)     --输出:  02015

string.format("%+04d", -2015)   --输出:  -2015

string.format("%+04d", 2015)   --输出:  +2015

string.format("%.5f", math.pi)   --输出:  3.14159

string.format("%.8f", 0.123456789)   --输出:  0.12345679 (这里可以看到第八位变成了9而不是8，其实是做了一个四舍五入操作)

string.format("%.4s", "canglang")   --输出:  cang

string.format("%8.4s", "canglang")   --输出:        cang

```



## 引用

* Lua程序设计-4版本

* ![OpenResty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/content/lua/main.html)
