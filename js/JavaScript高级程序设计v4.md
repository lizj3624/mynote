# JavaScript高级程序设计-4版

## JavaScript实现

1. 核心(ECMAScript)
它主要包含这门语言的如下部分

* 语法

* 类型

* 语句

* 关键字

* 保留字

* 操作符

* 全局对象

最广泛的ECMA-262第6版，俗称ES6, 支持了类、模板、迭代器、生成器、箭头函数、期约、反射、代理和众多新的数据类型。
2019年发布了ECMA-262第10版，ES9

2. 文档对象模型(DOM)
文档对象模型(DOM)是应用编程接口(API)，用于在HTML中使用扩展的XML。DOM将整个页面抽象为一组分层节点。HTML或者XML页面的每个组成部分都是一种节点，包含不同数据。
DOM通过创建表示文档的树，让开发者可以随心所欲的控制页面的内容和结构，使用DOM API可以轻松的删除、添加、替换、修改节点。

3. 浏览器对象模型(BOM)
  浏览器对象模型(BOM) API用于支持访问和操作浏览器的窗口。使用BOM，开发者可以操控浏览器显示页面之外的部分。

  ![js](https://github.com/lizj3624/mynote/blob/master/js/pictures/js-all.jpg)

4. HTML嵌入JavaScript
    将JS插入到HTML的主要方法是使用`<script>`元素。这个元素有8个属性

* `async`：可选，表示应该立即开始下载脚本，但不能阻止其他页面动作，比如下载资源或者等待其他脚本加载。只对外脚本文件有效。
* `charset`：可选。使用src属性指定的代码字符集。
* `crossorigin`：可选。配置相关请求的CORS设置。

* `defer`:可选。表示脚本可以延迟到文档完全被解析和显示后再执行。
* `intergrity`:可选。
* `language`：废弃
* `src`：可选，执行代码的外部文件
* `type`: 可选

```javascript
# 嵌入代码
<script>
    function sayHi() {
        console.log("Hi!");
    }
</script>

# 嵌入执行文件
<script src="example.js"></script>
```

![图片](https://github.com/lizj3624/mynote/blob/master/js/pictures/js-dom.jpg)

## 语言基础

### 语法

1、区分大小写，无论是变量、函数名还是操作符。

2、标识符

   1）第一个字符必须是字母，下划线(_)或者美元符$

   2）剩下的其他字符可以使字母、下划线(_)、美元符$或者数字

   3）驼峰书写

3、注释

```javascript
// 单行注释

/*
多行注释
*/
```

4、语句

```javascript
let sum = a + b
let diff = a - b;  分号不是强制的，但是最好有

## 语句块，可以{}，跟c语言类似
if (test) {
    console.log(test)
}
```

### 变量

1、var

声明和定义一个变量

```javascript
var message    #是未定义类型的变量

var message = "hi"，#定义一个字符串类型的变量
message = 10; //语法是没有问题的，但是不推荐
```

var声明提升，var声明的变量自动提升到函数作用域的顶部。

```javascript
function foo() {
    console.log(age);
    var age = 26;
}
foo();  // undefined

#相等于
function foo() {
    var age;
    console.log(age);
    age = 26; 
}
foo();  // undefined
```

2、let声明

`let`和`var`的差不多，但有着非常重要的区别，

1）最明显是：`leb`声明的范围是块作用域， `var`的声明范围是函数作用域。

```javascript
function myfunc() {
    if (true) {
        var name = 'Matt';    // var声明的name是函数作用域，if外可以访问name变量 
        console.log(name);    // Matt
    }
    console.log(name);        // Matt
}


function myfunc() {
    if (true) {
        let age = 26;        // let声明的age是块作用域，if外不可以访问age变量 
        console.log(age);    // age
    }
    console.log(age);        // ReferenceError: age 没有定义
}
```

2）`let`也不允许同一个块作用域中出现冗余声明

3）`let`声明的变量不会在作用域中被提升

4）使用`let`在全局作用域中声明的变量不会成为`window`对象的属性

```javascript
var name = 'Matt';
console.log(window.name); // 'Matt'
let age = 26;
console.log(window.age);  // undefined
```

5）`for`循环中的`let`声明

```javascript
for (var i = 0; i < 5; ++i) { 
    // 循环逻辑
}
console.log(i); // 5   var声明的变量回渗透到循环体外

for (let i = 0; i < 5; ++i) { 
    // 循环逻辑
}
console.log(i); // ReferenceError: i 没有定义，let 声明的循环遍历怎不会
```

3. `const`声明

    `const`的行为与`let`基本相同，唯一一个重要的区别是用它声明变量时必须同时初始化变量，且尝试修改 `const`声明的变量会导致运行时错误。作用域也是块作用域

    ```javascript
    // const 也不允许重复声明
    const name = 'Matt';
    const name = 'Nicholas'; // SyntaxError
    ```

    

### 数据类型

使用`typeof`可以查看变量类型

1、`Undefined`类型表示值未定义，只有一个值`undefined`，`var`和`let`声明的变量，但是初始化时，这个变量暂时被赋值给了`undefined`

2、`Null`类型表示`null`，也只有一个值`null`，表示一个空对象指针。

3、`Boolean`类型，`true`和`false`

4、`Number`表示整数和浮点数，整数可以用十进制、八进制、16进制表示

5、`String`数据类型表示零或多个16位 Unicode 字符序列。字符串可以使用双引号`"`、 单引号`'`或反引号(`)标示，因此下面的代码都是合法的:

```javascript
let firstName = "John";
let lastName = 'Jacob';
let lastName = `Jingleheimerschmidt`
```

字符串是不可改变的。

6、`Symbol`(符合)是 ECMAScript 6 新增的数据类型。符号是原始值，且符号实例是唯一、不可变的。 符号的用途是确保对象属性使用唯一标识符，不会发生属性冲突的危险。

7、`object`类型：ECMAScript 中的对象其实就是一组数据和功能的集合。对象通过 new 操作符后跟对象类型的名称 来创建。开发者可以通过创建 Object 类型的实例来创建自己的对象，然后再给对象添加属性和方法:

```javascript
    let o = new Object();
```

### 操作符

```javascript
// 一元运算
let age = 29
++age
--age

age++
age--
//注意区别

num = -num  //如果是整数，求负数

// 位运算
let num2 = ~num2;   //补数

let rusult = 25 & 3;  //与运算
let result = 25 | 3;  //或运算
let result = 25 ^ 3;  //异或运算
let result = result << 5;  //左移位
let result = result >> 5;   //有符号右移
let result = result >>> 5;  //无符号右移

//boolen
!false; //逻辑非
let result = true && true;  //逻辑与，短路操作
let result = true || true;  //逻辑或，短路操作

let result = 34 * 35;
let result = 34 / 11;

let result = 26 % 5; 

let result = 3 ** 2;  //指数

let result = 3 + 2;

let resutl = 4 - 2;

//关系操作符
> < <= >= 
== !=

//条件操作符
let max = (num1 > num2) ? num1 : num2;

// 赋值操作符
let num= 2;

num += 10;

//逗号
let num1 = 1, num2 = 2, num3 = 3;
```



### 语句

```javascript
// if条件
let j = 2;
if ( i > 5)
   j = 5;
else
   j = 3;

if (i > 5)
  ...
else if (i > 6)
  ...
else
  ...
  
// do-while语句
let i = 0;
do {
    i += 2;
} while (i > 10);

//while
let i = 0;
while (i < 10) {
    i += 2;
}

//for
let count = 10;
for (let i = 0; i < count; i++) {
    console.log(i);
}

// for-in
for (const propName in window) {
    document.write(propName);
}

for (const el of [2, 4, 6, 8]) {
    document.write(el);
}

//break, contiunue

//with
with (location) {
    let qs = search.substring(1);
}

//switch
switch (i) {
    case 25:
        ...
        break;
    case 35:
        ...
        break;
    default:
        ...;
}
```

### 函数

```javascript
function sayHi(name, message) {
    console.log("Hello " + name + ", " + message);
}
```

## 变量，作用域与内存

`Undefined，Null，Boolean，Number，String，Symbol`这六种原始值，按值传递。

引用值是保存在内存中的对象，按引用传递。

引用值可以动态添加、删除、修改其属性和方法

```javascript
let person = new Object();
person.name = "Nicholas";
console.log(person.name);
```

原始值不可以

```javascript
let name = "Nicholas"
name.age = 27;   //undefined
```

但是可以通过`new`创建原始值的对象

```javascript
let name2 = new String("Matt")
name2.age = 32
console.log(name2.age)   //32
```

函数参数按值传递

`typeof`确定类型

### 执行上下文与作用

1. 作用域链增强
2. 变量声明

### 垃圾回收

1. 标记清理
2. 引用计数
3. 性能
4. 内存管理

## 集合引用类型

### Object

```javascript
let person = new Object();
person.name = "Nicholas";
person.age = 29;

//对象字面量，属性的名字可以使字符串或者数值，不会调用构造函数
let persion = {
    name: "Nicholas",
    age: 29
};

//尽量用对象字面量表示法
```

### Array

```javascript
let colors = new Array();

let colors = new Array("red", "blue", "green");

let colors = new Array(3);

let names = new Array("Greg");

//字面量，不用调用Array的构造函数
let colors = ["red", "blue", "green"];

let names = [ ];

let values = [1, 2,];

//数组空位
const options = [,,,,,];

//数组索引，index从0开始
let colors = ["red", "blue", "green"];

alert(colors[0]);
alert(colors.length);

//判断是否数组
if (value instanceof Array) {
}

//迭代器
const a = {"foo", "bar", "baz", "qux"};

//keys, values, entries返回迭代器
const aKeys = Array.from(a.keys()); 
const aValues = Array.from(a.values());
const aEntries = Array.from(a.entries());

for (const [idx, element] of a.entries()) {
    alert(idx);
    alert(element);
};

//copyWithin()，fill()

//push数组末尾压入一项, pop弹出最后一项，相应的length都增减的变化
let colors = new Array();
let count = colors.push("red", "green");
let item = colors.pop();

//shift()，删除数组的第一项并返回它，然后数组长度减1，使用shift()和push()，可以把数组当成队列来使用
let colors = new Array();
let count = colors.push("red", "green");   //从尾部插入两个元素
alert(count);

count = colors.push("black");
alert(count);

let item = colors.shift();
alert(item);
alert(colors.length);

//排序, sort和reverse都返回调用他们的数组的引用
let values = [1, 2, 3, 3, 5];
values.reverse();   //逆序

let values = [0, 1, 5, 15];
values.sort();  //按照大小排序

values.sort(comapre)  //compare是一个排序函数

//concat，slice，splice
//indexOf从头开始搜索，返回第一项所在的位置
//lastIndexOf是从末尾开始，向前搜索
//includes() boolen
//find()，findIndex()
//every()，filter()，forEach()，map()，some()

//map
const m = new Map();

const m1 = new Map([
    ["key1","val1"],
    ["key2","val2"]
]);

m1.has("key1");  //boolen
m1.get("key1");
m1.delete("key1");
m1.size();

for (let pair of m.entries()) {
    alert(pair);
};

for (let key of m.keys()) {
    alert(key);
};

for (let val of m.values()) {
    alert(val);
};

//WeakMap
const wm = new WeakMap();

//set
const m = new Set();
const s1 = new Set(["val1", "val2", "val3"]);
s1.add();
s1.has(); //boolen
```

