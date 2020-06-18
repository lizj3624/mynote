## 一、Python基础
### 1、语句与语法
- 井号(#)表示是注释
- 换行(\n)是标准的行分隔符
- 反斜线`(\)`继续一行
- 分号(;)将两个语句链接在一行中
- 冒号(:)将代码块的头和体分开
- 语句(代码块)用缩进块的方式体现
- 不同的缩进深度分隔不同的代码块
- Python文件已模块的形式组织
### 2、标识符
- 第一个字符必须是字母或者下划线(_)
- 剩下的字符可以是字母和数字或下划线
- 大小写敏感
- Python用下划线作为前缀或者后缀指定特殊变量   
1）_xxx 不用`from module import *`
2）_xxx_ 系统定义名字
3）_xxx 类中的私有变量名
### 3、模块结构和布局
- 一种合理推荐布局格式   
1）起始行(Unix)   
2）模块文档   
3）模块导入   
4）变量定义    
5）类定义   
6）函数定义    
7）主程序   
```python
# /usr/bin/env python       # 起始行

"this is a test module"     # 模块文档

import sys                  # 模块导入
imprt os

debug = True                # （全局）变量定义

class FooClass (object):    # 类定义
    "Foo class"
    
    pass
    
def test():                 # 函数定义
    "test function"
    foo = FooClass()
    
    if debug:
       print 'ran test()'
       
if __name__ == '__main__':  #主程序
    test()
```
### 4、内存管理
- 变量无须事前先声明
- 变量无须指定类型
- 程序员不用关心内存管理
- 变量名会被""回收",GC
- del语句能够直接释放资源
- Python中，对象的类型和内存占用都是运行时确定的，Python仍然是一种解释性语言，解释器会根据语法和右侧的操作数来决定新对象的类型。

### 5、Python标准库
- [python3标准库](https://docs.python.org/3/library/index.html)
- [python2标准库](https://docs.python.org/2.7/library/index.html)

## 二、Python对象
- 每个Python对象都有身份、类型、值
- 对象可以有属性
- 标准类型   
1）Integer 整形      
2）Boolen 布尔型      
3）Long integer 长整型   
4）Floating point real number 浮点型   
5）Complex number 复数类型   
6）String 字符串   
7）List 列表   
8）Tuple 元组   
9）Dictionary 字典   
- 其它内建类型   
1）类型   
2）Null对象(None)    
3）文件   
4）集合/固定集合   
6）函数/方法   
7）模块   
8）类   
- `None`是Python的Null对象，没有属性，总是布尔值`False`
- 对象布尔值是`False`   
1）`None`   
2）`False`      
3）所有值为`0`的整数以及`0`   
4）`0.0` 或者`0L` 或者 `0.0+0.0j`   
5）`""`空字符串   
6）`[]` 空列表   
7）`()` 空元组   
8）`{}` 空字典   

## 三、数字
### 1、整形
- 布尔 True False
- 整形 四个字节或者八个字节
- 长整型 四个字节或者八个字节
- 以后整形和长整型统一
### 2、双精度的浮点数
- 占8个字节
### 3、复数
### 4、操作符
- `+ - * / % **`   
- `~ & | ^ << >>`
- 取反   
1）在计算机里面,负数是以补码存储的   
2）原码求补码:取反,+1   
3）补码求原码:取反,+1   
4）取反操作是在原码上进行的!   
5）实际的计算结果: `~4 = -5, ~-5 = 4`   
   4的原码: `0000 0100`    
   取反得到: `1111 1011`, 观察符号,是负数,因为负数以补码存储的,所以问题转化为:     
   某个数x的补码是`1111 1011`,求x的值(由补码求原码)     
   取反: `0000 0100`     
   `+1: 0000 0101 = 5`, 加上标点符号(负号) 得到结果: -5    

   求 ~-5,同理用八进制表示-5:     
   因为-5是负数,所以它是以5的补码表示的,所以转化为已知5的补码,求对应的原码,然后在取反.      
   5补码: `0000 0101`,     
   取反: `1111 1010`    
   `+1: 1111 1011`, 得到原码     
   取反: `0000 0100 = 4` ,加上标点负号(正号)得到结果:4
## 四、序列
### 1、序列包含字符串、列表、元组
- 操作符   
1）`in、not in`   
2）连接符 `+`   
3）重复操作符 `*`   
4）切片 `[] [:] [::]`   
   a）`sequeuece[index]`，index可以是正值 `0 <= index <= len(sequeuece) - 1`   
   b）index可以是负值，`-len(sequeuece) <= index <= -1`，正负区别就是序列起点不同    
   c）`sequeuece[start_index:end_index]`，可以从start_index到end_index(不包含end_index)切分sequeuece    
   d）按照步长切分`s[::-1]` 翻转 `s[::2]` 隔一个取一个
### 2、字符串
- 字符串是单引号('')或者单双号("")包含的字符集合，Python没有字符数据结构   
- 字符串不可分割
- 字符串 `%`输出
- 三引号的字符串可以包含换行符或者tab的特殊符号，三引号可以是`'''`或者`"""`
- 愿意输出或者unicode  `r'abd' u'abd'`
- Python字符串不是以`NUL`或者`""\0"`结束
### 3、列表
- 列表是用`[]`表示，可以包含Python定义各种数据类型，成员可以是不同数据类型    `a = [123, 'abc', 1.2]`   
- 可以通过切片或者索引访问
- 可以通过索引或者索引范围更新，也可以通过append追加
- del 删除
- 切片操作
- 列表解析   
```python
>>> [i * 2 for i in [8, -2, 5]]
[16, -4, 10]
```
- 库函数   
1）list()   
2）tuple()   
3）sort()、extend()、reverse()都是就地变更
### 4、元组
- 元组是跟列表非常相近的一种容器类型，元组使用()表示，与列表区别是元组不可变类型，可以做字典的key，处理对象时，默认是元组类型
- 用()表示，如果只有一个元素时元组分隔符中一定要有逗号（,）
- 切片操作
## 五、映射与集合类型
### 1、字典
- 字典是键值对(key-value)，key必须是固定不可变，可哈希的，字典都是无序的，用`{}`
- 定义字典
```python
>>> dict1 = {}
>>> dict2 = {'name':'earth', 'port': 80}
>>> fdict = dict((['x', 1], ['y', 2]))
>>> ddict = {}.fromkeys(('x', 'y'), -1)
```
- 访问字典中的值
```python
for key in dict2:
    print 'key = %s, v = %s' % (key, dict2[key])
```
- 修改字典中的值
```python
dict2['name'] = 'venus' #x修改
dict2['arch'] = 'sunos' #新增
```
- 删除字典元素以及字典
```python
del dict2['name']
dict2.clear()
del dict2
dict2.pop('name')
```
- 字典中键   
1）不允许一个键对应多个值，作为键必须不可变    
2）键必须是可哈希的
### 2、集合
- 集合对象是一组无序排列的可哈希的值，分为可变集合和不变集合
- 定义集合
```python
>>> s = set('cheeseshop')
>>> s = frozenset('bookshop')
```
- 操作符   
1）`in not in`   
2）`!= ==` 等不等价   
3）`< <=，> >=` 子集 超集   
4）联合`|`    
5）交集`&`   
6）差补集`-`，difference()   
7）对称差分`^`   

## 六、条件与循环语句
### 1、if 条件
```python
if condition:
    do_something
elif condition:
    do_something
else:
   do_something
```
- 三元操作符 `x if c else y`
### 2、 while
```python
count = 0
while (count < 9):
    print 'the index is ', count
    count += 1    
```
### 3、for
```python
#字符串迭代 不常用，迭代字符
for eachLetter in 'Name':
    print 'current letter: ', eachLetter
    
# 迭代序列
nameList = ['Walter', 'Nicole', 'Steven', 'Henry']
for eachName in nameList:
    print eachName, "Lim"
    
# 迭代索引
nameList = ['Walter', 'Nicole', 'Steven', 'Henry']
for nameIndex in range(len(nameList)):
    print "Lim, ", nameList[nameIndex]
    
# 迭代元素和索引
nameList = ['Walter', 'Nicole', 'Steven', 'Henry']
for nameIndex， name in enumerate(nameList):
    print nameIndex, name     
```
### 4、迭代器    
for循环访问迭代器和访问序列的方法差不多，唯一的区别是for语句会多做一些额外的事，
迭代器对象有个next()，迭代器调用完后会抛出异常，for 语句在内部调用next()并捕获异常
### 5、range
range(start, end, step = 1)
```python
range(2, 19, 3)
[2, 5, 8, 11, 14, 17]
```
### 6、break continue pass
### 7、迭代器
```python
my = ("12", 4, 7)
for each in my:
    do_something

fetch = iter(my)
whiel True:
    try
        i = fetch.next()
    except StopItration:
       break
    do_something()   
```
循环删除迭代器元素时要注意    
iter(obj)
### 8、列表解析
- 列表解析语法
```python
[expr for iter_var in iterable]
>>> [x ** 2 for x in range(6)]
[0, 1, 4, 9, 16, 25]

#condtion if
[expr for iter_var in iterable if cond_expr]
>>> [x for x in seq if x % 2]
[11, 9, 9, 9, 23, 9, 7, 11]

>>> [(x+1, y+1) for x in range(3) for y in range(5)]
[(1,1), (1, 2), (1, 3), (1, 4), (1, 5), (2,1), (2, 2), (2, 3), (2, 4), (2, 5),
(3,1), (3, 2), (3, 3), (3, 4), (3, 5),]
```
### 9、生成器表达式
- 语法
```python
(expr for iter_var in iterable if cond_expr)
```
- 列表解析的一个不足就是必要生产所有的数据，用以创建整个列表，这样可能对大量数据的迭代器有负面效应。
- 生成器表达式并不是真正创建数字列表，而是返回一个生产器，这个生产器在每次计算出一个条目后，把这个条目''产生'（yield）处理，使用了“延时计算”，所有在内存上更有效
```python
>>> sum(len(word) for line in data for word in line.split())
408

#计算文件中最长的行
#常用方法
f = open('/etc/motd', 'r')
longest = 0
while True:
    linelen = len(f.readline().strip())
    if not linelen: break
    if linelen > longest:
        longest = linelen
f.close()
return longest

#列表解析
f = open('/etc/motd', 'r')
longest = 0
allLines = [x.strip() for x in f.readlines()]
f.close()
for line in allLines:
    linelen = len(line)
    if not linelen: break
    if linelen > longest:
        longest = linelen
return longest

#改进列表解析
f = open('/etc/motd', 'r')
allLines = [len(x.strip()) for x in f]
f.close
return max(allLines)

#迭代器表达式
f = open('/etc/motd', 'r')
longest = max(len(x.strip()) for x in f)
f.close()
return longest
```
- [Python关键字yield的解释](https://pyzh.readthedocs.io/en/latest/the-python-yield-keyword-explained.html)
## 七、文件和输入输出
### 1、文件对象
```python
fp = open('/etc/mote')   #以读方式打开
fp = open('test', 'w')   #以写方式打开
fp = open('data', 'r+')  #以读方式打开
fp = open(r'c:\io.sys', 'rb') #以二进制方式打开
```
### 2、输入
- `read()`方法用来直接读取字节到字符串中，最多读取给定数目个字节
- `readline()`方法读取打开文件的一行(读取下个行结束符之前的所有字节)，然后整行，包含航结束符。作为字符串返回
- `readlines()`它会读取所有(剩余的)行然后把它们作为一个字符串列表返回。
### 3、输出
- `write()`它把含有文本数据或二进制数据块的字符串写入文件中去
- `writelines()`将字符串列表写入文件中，行结束符并不会被自动加入
- 当使用输入方法`read()`或者`readlines()`从文件中读取行时，python并不会删除行结束符
```python
f = open('myfile', 'r')
data = [line.strip() for line in f.readlines()]
f.close()
```
- 文件迭代
```python
for eachLine in f:
    ：
```
## 八、错误与异常
### 1、检测与处理异常
```python
try
    f = open('blan', 'r')
except IOError, e:
    print 'could not open file:', e
    
### 多个except
try
    f = open('blan', 'r')
except IOError, e:
    print 'could not open file:', e
except TypeError, e:
    print 'could not open file:', e    
    
### finaly    
try
    ...
finaly
    ...
```
### 2、with
```python
with context_expr [as var]:
    ...
```

### 3、raise
### 4、断言
```python
assert 1 == 1
```
## 九、函数和函数式编程
### 1、函数
```python
def function_name(arge):
    ...dosomething
```
### 2、装饰器
- [python 装饰器详解](https://www.cnblogs.com/cicaday/p/python-decorator.html)
## 十、模块
### 1、模块和文件
- 模块支持从逻辑上组织Python代码
- 一个名称空间就是一个从名称到对象的关系映射集合
- 默认搜索路径`PYTHONPATH`，`sys.path`
### 2、导入模块
```python
import module1
import module2
import module3
```
- 推荐import 模块顺序   
1）Python标准模块    
2）Python第三方模块    
3）应用程序自定义模块   
然后使用一个空格分割这三类模块的导入部分    
- `from-import`语句    
在指定你的模块中导入指定模块的属性    
```python
from module import name1 [, name2...]

##不推荐 
from module import *
```
- 一个模块只被加载一次，无论它被导入多少次
### 3、包
包是一个有层次的文件目录结构，他定义了一个由模块和子包组成的Python应用执行环境    
```python
Phone/
    __init__.py
    common_util.py
    Voicedta/
        __init__.py
        Pots.py
        Isdn.py
    Fax/
        __init__.py
        G3.py
    Mobile
        __init__.py
        Analog.py
        Digital.py
    Pager/
        __init__.py
        Numeric.py
```
Phone是顶层的包，Voicedta等是它的子包，导入子包    
```python
import Phone.Mobile.Analog
Phone.Mobile.Analog.dial()

## from-import
from Phone import Mobile
Mobile.Analog.dial()
```
- 源代码编码
```python
#!/usr/bin/env python
# -*- coding: UTF-8 -*-
```
## 十一、面向对象编程
### 1、类
```python
class ClassName(object):
    'class doc'
    class_sult
    
class HotelRoomCalc(object):
    'Hotel room rate calculator'
    
    def __init__(self, rt, salts=0.085, rm=01):
        'HotelRoomCalc doc'
        self.salesTax = rm
        self.roomTax = rm
        self.roomRate = rt
        
    def calcTotal(self. days=1):
        'Calculate total'
        daily = round((self.roomRate*(1 + self.roomTax + self.salesTax)), 2)
        return float(days) * daily    
```
- `__init__()`应当返回`None`
- 实例属性和类属性    
1）类属性跟类有关，想静态变量一样引用    
2）实例属性跟实例对象有关，必须实例对象调用   
3）类和实例都是名字空间，类是类属性的名字空间，实例是实例属性空间    
4）类属性不能能被实例修改，实例可以覆盖类属性
- 静态方法和类方法
### 2、绑定核方法调用
- 方法仅仅是类内部定义的函数
- 方法只有在其所属的类拥有实例时，才能被调用
- 任何一个方法定义中第一个参数是变量`self`
- `self`变量用于在类实例方法中引用方法所绑定的实例，类方法必须加上`self`
- 调用非绑定方法    
一个主要的场景是：派生一个子类，而且需要覆盖父类的方法
```python
class EmplAddrBookEntry(AddBookEntry):
    'Employee Address Book Entry class'
    def __init__(self, nm, ph, id, em):
        AddrBookEntry.__init__(self, nm, ph)  ##可以用super代替
        ##super(EmplAddrBookEntry, self).__init__(nm, ph)
        self.empid = id
        self.email = em
```
### 3、继承
```python
class p(object):
   def foo(self):
       print 'Hi, I am P-foo()'
       
class C(P):
    def foo(self):
        super(C, self)
        print 'Hi, I am C-foo'
```
### 4、私有化
- 双下划线(__)    
为类的属性和方法提供私有性    
1）`__xxx__` 系统定义名字    
2）`__xxx` 类中的私有变量名    
- 下划线(_)   
`_xxx` 不能用`from module import *`导入 

### 5、导入
```python
if __name__ == '__main__'
```
如果相等时(main)，执行main内代码，否则只是打算导入这个脚本
