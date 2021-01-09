# lua程序设计
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
```

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


