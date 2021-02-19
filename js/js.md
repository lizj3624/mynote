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

