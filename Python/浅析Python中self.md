[toc]
### 浅析Python中self
#### self的用法
`Python`要求，类（`class`）方法（构造方法`__init__`和实例方法）中至少要包含一个参数，但并没有规定此参数的名称（完全可以叫任意参数名），之所以将类方法的第一个参数命名为`self`，只是`Python`程序员约定俗成的一种习惯，这会使程序具有更好的可读性。   

如果你接触过其他面向对象的编程语言（例如 `C++`），其实 Python 类方法中的`self`参数就相当于`C++`中的`this`指针。
也就是说，同一个类可以产生多个对象，当某个对象调用类方法时，该对象会把自身的引用作为第一个参数自动传给该方法，换句话说，`Python`会自动绑定类方法的第一个参数指向调用该方法的对象。如此，`Python`解释器就能知道到底要操作哪个对象的方法了。

对于构造方法来说，`self`参数（第一个参数）代表该构造方法`__init__`正在初始化的对象。
因此，程序在调用实例方法和构造方法时，不需要为第一个参数传值。例如，更改前面的`Dog` 类，如下所示：
```python
class Dog:
    def __init__(self):
        print(self,"在调用构造方法")
    # 定义一个jump()方法
    def jump(self):
        print(self,"正在执行jump方法")
    # 定义一个run()方法，run()方法需要借助jump()方法
    def run(self):
        print(self,"正在执行run方法")
        # 使用self参数引用调用run()方法的对象
        self.jump()
dog1 = Dog()
dog1.run()
dog2 = Dog()
dog2.run()
```

**注意**，当`Python` 对象的一个方法调用另一个方法时，不可以省略`self`。也就是说，将上面的 `run()`方法改为如下形式是不正确的：
```python
# 定义一个run()方法，run()方法需要借助jump()方法
def run():
    #省略self，代码会报错
    self.jump()
    print("正在执行run方法")
    
class InConstructor :
    def __init__(self) :
        # 在构造方法里定义一个foo变量（局部变量）
        foo = 0
        # 使用self代表该构造方法正在初始化的对象
        # 下面的代码将会把该构造方法正在初始化的对象的foo实例变量设为6
        self.foo = 6
# 所有使用InConstructor创建的对象的foo实例变量将被设为6
print(InConstructor().foo) # 输出6
```
#### Python中为何要有self
在类的代码（函数）中，需要访问当前的实例中的变量和函数的，即，访问`Instance`中的：
- 调用对应的变量（property)：`self.ProperyyName`
- 调用对应函数或者方法（function）：`self.function()`
- 需要访问实例的变量和调用实例的函数，当然需要对应的实例`Instance`对象本身`self`
- `Python`中就规定好了，函数的第一个参数，就必须是实例对象本身，并且建议，约定俗成，把其名字写为`self`   

而如果没有用到`self`，即代码中，去掉`self`后，那种写法所使用到的变量，实际上不是你所希望的，不是真正的实例中的变量和函数，而是的访问到了其他部分的变量和函数了。**甚至会由于没有合适的初始化实例变量，而导致后续无法访问的错误。**    
**如果没有在`__init__`中初始化对应的实例变量的话，导致后续引用实例变量会出错**
```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
#注：此处全局的变量名，写成name，只是为了演示而用
#实际上，好的编程风格，应该写成gName之类的名字，以表示该变量是Global的变量
name = "whole global name";
 
class Person:
    def __init__(self, newPersionName):
        #self.name = newPersionName;
         
        #1.如果此处不写成self.name
        #那么此处的name，只是__init__函数中的局部临时变量name而已
        #和全局中的name，没有半毛钱关系
        name = newPersionName;
         
        #此处只是为了代码演示，而使用了局部变量name，
        #不过需要注意的是，此处很明显，由于接下来的代码也没有利用到此处的局部变量name
        #则就导致了，此处的name变量，实际上被浪费了，根本没有利用到
 
    def sayYourName(self):
        #此处由于找不到实例中的name变量，所以会报错：
        #AttributeError: Person instance has no attribute 'name'
        print 'My name is %s'%(self.name);
 
def selfAndInitDemo():
    persionInstance = Person("crifan");
    persionInstance.sayYourName();
     
###############################################################################
if __name__=="__main__":
    selfAndInitDemo();
```
从上述代码可见，由于在类的初始化（实例化）的`__init_`_函数中，没有给`self.name`设置值，使得实例中，根本没有`name`这个变量，导致后续再去访问`self.name`，就会出现`AttributeError`的错误了。    

#### 总结

- self在定义时需要定义，但是在调用时会自动传入。

- self的名字并不是规定死的，但是最好还是按照约定是用self

- self总是指调用时的类的实例。

- 定义类属性和类方式时，第一个参数要是`self`，不用`self`时可能引发错误


#### 引用
- [Python中：self和`__init__`的含义 + 为何要有self和`__init__`](https://www.crifan.com/summary_the_meaning_of_self_and___init___in_python_and_why_need_them/)
- [Python类中的self到底是干啥的](http://chown-jane-y.coding.me/2017/03/22/Python%E7%B1%BB%E4%B8%AD%E7%9A%84self%E5%88%B0%E5%BA%95%E6%98%AF%E5%B9%B2%E5%95%A5%E7%9A%84/)