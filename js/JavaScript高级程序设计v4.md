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

