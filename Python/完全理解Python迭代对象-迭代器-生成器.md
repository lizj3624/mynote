[toc]
### 完全理解Python迭代对象、迭代器、生成器
在了解Python的数据结构时，容器(container)、可迭代对象(iterable)、迭代器(iterator)、生成器(generator)、列表/集合/字典推导式(list,set,dict comprehension)众多概念参杂在一起，难免让初学者一头雾水，我将用一篇文章试图将这些概念以及它们之间的关系捋清楚。    
#### 容器(container)
容器是一种把多个元素组织在一起的数据结构，容器中的元素可以逐个地迭代获取，可以用`in, not in`关键字判断元素是否包含在容器中。通常这类数据结构把所有的元素存储在内存中（也有一些特例，并不是所有的元素都放在内存，比如迭代器和生成器对象）在Python中，常见的容器对象有：

- list, deque, ....
- set, frozensets, ....
- dict, defaultdict, OrderedDict, Counter, ....
- tuple, namedtuple, …
- str    
容器比较容易理解，因为你就可以把它看作是一个盒子、一栋房子、一个柜子，里面可以塞任何东西。从技术角度来说，当它可以用来询问某个元素是否包含在其中时，那么这个对象就可以认为是一个容器，比如`list，set，tuples`都是容器对象：
```python
>>> assert 1 in [1, 2, 3]      # lists
>>> assert 4 not in [1, 2, 3]
>>> assert 1 in {1, 2, 3}      # sets
>>> assert 4 not in {1, 2, 3}
>>> assert 1 in (1, 2, 3)      # tuples
>>> assert 4 not in (1, 2, 3)

# 询问某元素是否在dict中用dict的中key：
>>> d = {1: 'foo', 2: 'bar', 3: 'qux'}
>>> assert 1 in d
>>> assert 'foo' not in d  # 'foo' 不是dict中的元素

# 询问某substring是否在string中：
>>> s = 'foobar'
>>> assert 'b' in s
>>> assert 'x' not in s
>>> assert 'foo' in s 
```
尽管绝大多数容器都提供了某种方式来获取其中的每一个元素，但这并不是容器本身提供的能力，而是可迭代对象赋予了容器这种能力，当然并不是所有的容器都是可迭代的，比如：[Bloom filter](https://zh.wikipedia.org/wiki/%E5%B8%83%E9%9A%86%E8%BF%87%E6%BB%A4%E5%99%A8)，虽然Bloom filter可以用来检测某个元素是否包含在容器中，但是并不能从容器中获取其中的每一个值，因为Bloom filter压根就没把元素存储在容器中，而是通过一个散列函数映射成一个值保存在数组中。

#### 可迭代对象(iterable)
刚才说过，很多容器都是可迭代对象，此外还有更多的对象同样也是可迭代对象，比如处于打开状态的files，sockets等等。但凡是可以返回一个迭代器的对象都可称之为可迭代对象，听起来可能有点困惑，没关系，先看一个例子：
```python
>>> x = [1, 2, 3]
>>> y = iter(x)
>>> z = iter(x)
>>> next(y)
1
>>> next(y)
2
>>> next(z)
1
>>> type(x)
<class 'list'>
>>> type(y)
<class 'list_iterator'>
```
这里x是一个可迭代对象，可迭代对象和容器一样是一种通俗的叫法，并不是指某种具体的数据类型，list是可迭代对象，dict是可迭代对象，set也是可迭代对象。y和z是两个独立的迭代器，迭代器内部持有一个状态，该状态用于记录当前迭代所在的位置，以方便下次迭代的时候获取正确的元素。迭代器有一种具体的迭代器类型，比如`list_iterator，set_iterator`。可迭代对象实现了`__iter__`方法，该方法返回一个迭代器对象。
```python
x = [1, 2, 3]
for elem in x:
    ...
```
反编译该段代码，你可以看到解释器显示地调用`GET_ITER`指令，相当于调用`iter(x)，FOR_ITER`指令就是调用`next()`方法，不断地获取迭代器中的下一个元素，但是你没法直接从指令中看出来，因为他被解释器优化过了。
当运行代码：
```python
>>> import dis
>>> x = [1, 2, 3]
>>> dis.dis('for _ in x: pass')
  1           0 SETUP_LOOP              14 (to 17)
              3 LOAD_NAME                0 (x)
              6 GET_ITER
        >>    7 FOR_ITER                 6 (to 16)
             10 STORE_NAME               1 (_)
             13 JUMP_ABSOLUTE            7
        >>   16 POP_BLOCK
        >>   17 LOAD_CONST               0 (None)
             20 RETURN_VALUE
```
#### 迭代器(iterator)
那么什么迭代器呢？它是一个带状态的对象，他能在你调用`next()`方法的时候返回容器中的下一个值，任何实现了`__iter__`和`__next__()`（`python2`中实现`next()`）方法的对象都是迭代器，`__iter__`返回迭代器自身，`__next__`返回容器中的下一个值，如果容器中没有更多元素了，则抛出`StopIteration`异常，至于它们到底是如何实现的这并不重要。
所以，迭代器就是实现了工厂模式的对象，它在你每次你询问要下一个值的时候给你返回。有很多关于迭代器的例子，比如itertools函数返回的都是迭代器对象。
```python
>>> from itertools import cycle
>>> colors = cycle(['red', 'white', 'blue'])
>>> next(colors)
'red'
>>> next(colors)
'white'
>>> next(colors)
'blue'
>>> next(colors)
'red'

#为了更直观地感受迭代器内部的执行过程，我们自定义一个迭代器，以斐波那契数列为例：
class Fib:
    def __init__(self):
        self.prev = 0
        self.curr = 1

    def __iter__(self):
        return self

    def __next__(self):
        value = self.curr
        self.curr += self.prev
        self.prev = value
        return value

>>> f = Fib()
>>> list(islice(f, 0, 10))
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```
`Fib`既是一个可迭代对象（因为它实现了`__iter__`方法），又是一个迭代器（因为实现了`__next__`方法）。实例变量`prev`和`curr`用户维护迭代器内部的状态。每次调用next()方法的时候做两件事：
为下一次调用`next()`方法修改状态
为当前这次调用生成返回结果
迭代器就像一个懒加载的工厂，等到有人需要的时候才给它生成值返回，没调用的时候就处于休眠状态等待下一次调用。
```python
#判断是否迭代器
>>> from collections import Iterable
>>> isinstance([], Iterable)
True
>>> isinstance({}, Iterable)
True
>>> isinstance('abc', Iterable)
True
>>> isinstance((x for x in range(10)), Iterable)
True
>>> isinstance(100, Iterable)
False

# 文件是可迭代对象，也是迭代器
f = open('housing.csv')
from collections import Iterator
from collections import Iterable
 
print(isinstance(f,Iterator))
print(isinstance(f,Iterable))
 
True
True

#for循环可以自动处理StopIteration，while需要需要捕捉StopIteration
for x in [1, 2, 3, 4, 5]:
    pass
    
# 实际上完全等价于
# 首先获得Iterator对象:
it = iter([1, 2, 3, 4, 5])
# 循环:
while True:
    try:
        # 获得下一个值:
        x = next(it)
    except StopIteration:
        # 遇到StopIteration就退出循环
        break
```
#### 生成器(generator)
生成器算得上是Python语言中最吸引人的特性之一，生成器其实是一种特殊的迭代器，不过这种迭代器更加优雅。它不需要再像上面的类一样写`__iter__()`和`__next__()`方法了，只需要一个`yiled`关键字。 生成器一定是迭代器（反之不成立），因此任何生成器也是以一种懒加载的模式生成值。用生成器来实现斐波那契数列的例子是：
```python
def fib():
    prev, curr = 0, 1
    while True:
        yield curr
        prev, curr = curr, curr + prev

>>> f = fib()
>>> list(islice(f, 0, 10))
[1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```
`fib`就是一个普通的python函数，它特殊的地方在于函数体中没有return关键字，函数的返回值是一个生成器对象。当执行`f=fib()`返回的是一个生成器对象，此时函数体中的代码并不会执行，只有显示或隐示地调用next的时候才会真正执行里面的代码。

生成器在Python中是一个非常强大的编程结构，可以用更少地中间变量写流式代码，此外，相比其它容器对象它更能节省内存和CPU，当然它可以用更少的代码来实现相似的功能。现在就可以动手重构你的代码了，但凡看到类似：
```python
def something():
    result = []
    for ... in ...:
        result.append(x)
    return result
    
#都可以用生成器函数来替换：
def iter_something():
    for ... in ...:
        yield x
```
　通过列表生成式，我们可以直接创建一个列表，但是，受到内存限制，列表容量肯定是有限的，而且创建一个包含100万个元素的列表，不仅占用很大的存储空间，如果我们仅仅需要访问前面几个元素，那后面绝大多数元素占用的空间都白白浪费了。

　　所以，如果列表元素可以按照某种算法推算出来，那我们是否可以在循环的过程中不断推算出后续的元素呢？这样就不必创建完整的list，从而节省大量的空间，在Python中，这种一边循环一边计算的机制，称为生成器：generator

　　生成器是一个特殊的程序，可以被用作控制循环的迭代行为，python中生成器是迭代器的一种，使用yield返回值函数，每次调用yield会暂停，而可以使用next()函数和send()函数恢复生成器。

　　生成器类似于返回值为数组的一个函数，这个函数可以接受参数，可以被调用，但是，不同于一般的函数会一次性返回包括了所有数值的数组，生成器一次只能产生一个值，这样消耗的内存数量将大大减小，而且允许调用函数可以很快的处理前几个返回值，因此生成器看起来像是一个函数，但是表现得却像是迭代器。    
要创建一个`generator`，有很多种方法，第一种方法很简单，只有把一个列表生成式的[]中括号改为`（）`小括号，就创建一个`generator`
```python
#列表生成式
lis = [x*x for x in range(10)]
print(lis)
#生成器
generator_ex = (x*x for x in range(10))
print(generator_ex)
 
结果：
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
<generator object <genexpr> at 0x000002A4CBF9EBA0>

#生成器
generator_ex = (x*x for x in range(10))
for i in generator_ex:
    print(i)
     
结果：
0
1
4
9
16
25
36
49
64
81

##用for循环调用generator时，发现拿不到generator的return语句的返回值
def fib(max):
    n,a,b =0,0,1
    while n < max:
        yield b
        a,b =b,a+b
        n = n+1
    return 'done'
for i in fib(6):
    print(i)
     
结果：
1
1
2
3
5
8

# while循环时，需要捕捉生产器的StopIteration错误
def fib(max):
    n,a,b =0,0,1
    while n < max:
        yield b
        a,b =b,a+b
        n = n+1
    return 'done'
g = fib(6)
while True:
    try:
        x = next(g)
        print('generator: ',x)
    except StopIteration as e:
        print("生成器返回值：",e.value)
        break
 
 
结果：
generator:  1
generator:  1
generator:  2
generator:  3
generator:  5
generator:  8
生成器返回值： done
```
#### 生成器表达式(generator expression)
生成器表达式是列表推倒式的生成器版本，看起来像列表推导式，但是它返回的是一个生成器对象而不是列表对象。
```python
>>> a = (x*x for x in range(10))
>>> a
<generator object <genexpr> at 0x401f08>
>>> sum(a)
285

>>> # 列表解析生成列表
>>> [ x ** 3 for x in range(5)]
[0, 1, 8, 27, 64]
>>>
>>> # 生成器表达式
>>> (x ** 3 for x in range(5))
<generator object <genexpr> at 0x000000000315F678>
>>> # 两者之间转换
>>> list(x ** 3 for x in range(5))
[0, 1, 8, 27, 64]
```
#### 总结
- 容器是一系列元素的集合，`str、list、set、dict、file、sockets`对象都可以看作是容器，容器都可以被迭代（用在`for，while`等语句中），因此他们被称为可迭代对象。
- 可迭代对象实现了`__iter__`方法，该方法返回一个迭代器对象。
- 迭代器持有一个内部状态的字段，用于记录下次迭代返回值，它实现了`__next__`和`__iter__`方法，迭代器不会一次性把所有元素加载到内存，而是需要的时候才生成返回结果。
- 生成器是一种特殊的迭代器，它的返回值不是通过`return`而是用`yield`。
#### 引用
- [完全理解Python迭代对象、迭代器、生成器](https://foofish.net/iterators-vs-generators.html)
- [python 生成器和迭代器有这篇就够了](https://www.cnblogs.com/wj-1314/p/8490822.html)
- [Iterables vs. Iterators vs. Generators](https://nvie.com/posts/iterators-vs-generators/)
- [对Python迭代的深入研究](http://kuanghy.github.io/2016/05/18/python-iteration)