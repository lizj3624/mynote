### python的异常处理
异常处理在任何一门编程语言里都是值得关注的一个话题，良好的异常处理可以让你的程序更加健壮，清晰的错误信息更能帮助你快速修复问题。在Python中，和部分高级语言一样，使用了`try/except/finally`语句块来处理异常，如果你有其他编程语言的经验，实践起来并不难。
#### 什么是异常
- 错误    
从软件方面来说，错误是语法或是逻辑上的。错误是语法或是逻辑上的。
语法错误指示软件的结构上有错误，导致不能被解释器解释或编译器无法编译。这些些错误必须在程序执行前纠正。当程序的语法正确后，剩下的就是逻辑错误了。逻辑错误可能是由于不完整或是不合法的输入所致；在其它情况下，还可能是逻辑无法生成、计算、或是输出结果需要的过程无法执行。这些错误通常分别被称为域错误和范围错误。当python检测到一个错误时，python解释器就会指出当前流已经无法继续执行下去。这时候就出现了异常。

- 异常    
对异常的最好描述是：它是因为程序出现了错误而在正常控制流以外采取的行为。这个行为又分为两个阶段：首先是引起异常发生的错误，然后是检测（和采取可能的措施）阶段。第一阶段是在发生了一个异常条件（有时候也叫做例外的条件）后发生的。只要检测到错误并且意识到异常条件，解释器就会发生一个异常。引发也可以叫做触发，抛出或者生成。解释器通过它通知当前控制流有错误发生。python也允许程序员自己引发异常。无论是python解释器还是程序员引发的，异常就是错误发生的信号。当前流将被打断，用来处理这个错误并采取相应的操作。这就是第二阶段。对于异常的处理发生在第二阶段，异常引发后，可以调用很多不同的操作。可以是忽略错误（记录错误但不采取任何措施，采取补救措施后终止程序。）或是减轻问题的影响后设法继续执行程序。所有的这些操作都代表一种继续，或是控制的分支。关键是程序员在错误发生时可以指示程序如何执行。python用异常对象`(exceptionobject)`来表示异常。遇到错误后，会引发异常。如果异常对象并未被处理或捕捉，程序就会用所谓的回溯`（traceback）`终止执行

#### 异常处理
- 捕捉异常可以使用`try/except`语句
- 捕捉异常也可以使用`try/excpet/finally`语句
```python
# try/except
try:
    <语句>        #运行别的代码
except <名字>：
    <语句>        #如果在try部份引发了'name'异常
except <名字>，<数据>:
    <语句>        #如果引发了'name'异常，获得附加的数据
else: 
    <语句>        #如果没有异常发生
    
#  try/excpet/finally
try:
    <语句>
except <名字>：
    <语句>
finally:     #finally语句总是会被执行
    <语句>
    
def div(a, b):
    try:
        print(a / b)
    except ZeroDivisionError:
        print("Error: b should not be 0 !!")
    except Exception as e:
        print("Unexpected Error: {}".format(e))
    else:
        print('Run into else only when everything goes well')
    finally:
        print('Always run into finally block.')

# tests
div(2, 0)
div(2, 'bad type')
div(1, 2)

# Mutiple exception in one line
try:
    print(a / b)
except (ZeroDivisionError, TypeError) as e:
    print(e)

# Except block is optional when there is finally
try:
    open(database)
finally:
    close(database)

# catch all errors and log it
try:
    do_work()
except:    
    # get detail from logging module
    logging.exception('Exception caught!')
    
    # get detail from sys.exc_info() method
    error_type, error_value, trace_back = sys.exc_info()
    print(error_value)
    raise    
```
#### 总结如下
- `except`语句不是必须的，`finally`语句也不是必须的，但是二者必须要有一个，否则就没有`try`的意义了。
- `except`语句可以有多个，`Python`会按`except`语句的顺序依次匹配你指定的异常，如果异常已经处理就不会再进入后面的`except`语句。
- `except`语句可以以元组形式同时指定多个异常，参见实例代码。
- `except`语句后面如果不指定异常类型，则默认捕获所有异常，你可以通过`logging`或者`sys`模块获取当前异常。
- 如果要捕获异常后要重复抛出，请使用`raise`，后面不要带任何参数或信息。
- 不建议捕获并抛出同一个异常，请考虑重构你的代码。
- 不建议在不清楚逻辑的情况下捕获所有异常，有可能你隐藏了很严重的问题。
- 尽量使用内置的异常处理语句来替换`try/except`语句，比如`with`语句，`getattr()`方法。

#### 抛出异常 raise
如果你需要自主抛出异常一个异常，可以使用`raise`关键字，等同于`C#`和`Java`中的`throw`，其语法规则如下:
```python
raise NameError("bad name!")
```
`raise`关键字后面可以指定你要抛出的异常实例，一般来说抛出的异常越详细越好，`Python`在`exceptions`模块内建了很多的异常类型，通过使用`dir()`函数来查看`exceptions`中的异常类型，如下：
```python
import exceptions

print dir(exceptions)
# ['ArithmeticError', 'AssertionError'...]
```
当然你也可以查阅Python的文档库进行更详细的了解: [官方异常文档](https://docs.python.org/zh-cn/3/tutorial/errors.html)

#### 自定义异常类型
`Python`中自定义自己的异常类型非常简单，只需要要从`Exception`类继承即可(直接或间接)：
```python
class SomeCustomException(Exception):
    pass

class AnotherException(SomeCustomException):
    pass
```
一般你在自定义异常类型时，需要考虑的问题应该是这个异常所应用的场景。如果内置异常已经包括了你需要的异常，建议考虑使用内置的异常类型。比如你希望在函数参数错误时抛出一个异常，你可能并不需要定义一个`InvalidArgumentError`，使用内置的`ValueError`即可。

#### 经验案例
- 传递异常`re-raise Exception`    
捕捉到了异常，但是又想重新抛出它（传递异常），使用不带参数的`raise`语句即可：
```python
def f1():
    print(1/0)

def f2():
    try:
        f1()
    except Exception as e:
        raise  # don't raise e !!!

f2()
```
在`Python2`中，为了保持异常的完整信息，那么你捕获后再次抛出时千万不能在`raise`后面加上异常对象，否则你的`trace`信息就会从此处截断。以上是最简单的重新抛出异常的做法，也是推荐的做法。还有一些技巧可以考虑，比如抛出异常前你希望对异常的信息进行更新。
```python
def f2():
    try:
        f1()
    except Exception as e:
        e.args += ('more info',)
        raise
```
如果你有兴趣了解更多，建议阅读这篇[博客](http://www.ianbicking.org/blog/2007/09/re-raising-exceptions.html)   
`Python3`对重复传递异常有所改进，你可以自己尝试一下，不过建议还是遵循以上规则
- `Exception`和`BaseException`
当我们要捕获一个通用异常时，应该用`Exception`还是`BaseException`？我建议你还是看一下 官方文档说明，这两个异常到底有啥区别呢？ 请看它们之间的继承关系。
```python
BaseException
 +-- SystemExit
 +-- KeyboardInterrupt
 +-- GeneratorExit
 +-- Exception
      +-- StopIteration...
      +-- StandardError...
      +-- Warning...
```
从`Exception`的层级结构来看，`BaseException`是最基础的异常类，`Exception`继承了它。`BaseException`除了包含所有的`Exception`外还包含了`SystemExit，KeyboardInterrupt和GeneratorExit`三个异常。

由此看来你的程序在捕获所有异常时更应该使用`Exception`而不是`BaseException`，因为被排除的三个异常属于更高级别的异常，合理的做法应该是交给`Python`的解释器处理。
- `except Exception as e`和`except Exception, e`    
代码示例如下：
```python
try:
    do_something()
except NameError as e:  # should
    pass
except KeyError, e:  # should not
    pass
```
在`Python2`的时代，你可以使用以上两种写法中的任意一种。在`Python3`中你只能使用第一种写法，第二种写法已经不再支持。第一个种写法可读性更好，而且为了程序的兼容性和后期移植的成本，请你果断抛弃第二种写法。

- raise "Exception string"    
把字符串当成异常抛出看上去是一个非常简洁的办法，但其实是一个非常不好的习惯。
```python
if is_work_done():
    pass
else:
    raise "Work is not done!" # not cool
```
- 使用内置的语法范式代替`try/except`
`Python`本身提供了很多的语法范式简化了异常的处理，比如`for`语句就处理了的`StopIteration`异常，让你很流畅地写出一个循环。
`with`语句在打开文件后会自动调用`finally`并关闭文件。我们在写`Python 代码时应该尽量避免在遇到这种情况时还使用`try/except/finally`的思维来处理。
```python
# should not
try:
    f = open(a_file)
    do_something(f)
finally:
    f.close()

# should 
with open(a_file) as f:
    do_something(f)
```
再比如，当我们需要访问一个不确定的属性时，有可能你会写出这样的代码：
```python
try:
    test = Test()
    name = test.name  # not sure if we can get its name
except AttributeError:
    name = 'default'
```
其实你可以使用更简单的`getattr()`来达到你的目的。
```python
name = getattr(test, 'name', 'default')
```
#### 最佳实践
- 只处理你知道的异常，避免捕获所有异常然后吞掉它们。
- 抛出的异常应该说明原因，有时候你知道异常类型也猜不出所以然。
- 避免在catch语句块中干一些没意义的事情，捕获异常也是需要成本的。
- 不要使用异常来控制流程，那样你的程序会无比难懂和难维护。
- 如果有需要，切记使用finally来释放资源。
- 如果有需要，请不要忘记在处理异常后做清理工作或者回滚操作。

#### 引用
- [Python中的异常处理](https://segmentfault.com/a/1190000007736783)
