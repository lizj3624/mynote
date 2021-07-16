### 列表
列表是Python中最基本的数据结构，列表是最常用的Python数据类型，列表是一个数据的集合，集合内可以放任何数据类型，可对集合方便的增删改查操作。
#### 列表创建
```python
my_list = [] #空列表
my_list = ['abc', 123]
```
#### 列表的切片
一个完整的切片表达式包含两个`“:”`，用于分隔三个参数`(start_index、end_index、step)`，当只有一个`“:”`时，默认第三个参数`step=1`。    
- step：正负数均可，其绝对值大小决定了切取数据时的‘‘步长”，而正负号决定了“切取方向”，正表示“从左往右”取值，负表示“从右往左”取值。当step省略时，默认为1，即从左往右以增量1取值。**“切取方向非常重要！”“切取方向非常重要！”“切取方向非常重要！”，重要的事情说三遍！**
- start_index：表示起始索引（包含该索引本身）；该参数省略时，表示从对象“端点”开始取值，至于是从“起点”还是从“终点”开始，则由step参数的正负决定，step为正从“起点”开始，为负从“终点”开始。
- end_index：表示终止索引（不包含该索引本身）；该参数省略时，表示一直取到数据“端点”，至于是到“起点”还是到“终点”，同样由step参数的正负决定，step为正时直到“终点”，为负时直到“起点”。
```python
>>>a = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```
- 切取单个值
```python
>>>a[0]
>>>0
>>>a[-4]
>>>6
```
- 切取完整对象
```python
>>>a[:] #从左往右
>>> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>>a[::] #从左往右
>>> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>>a[::-1] #从右往左
>>> [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]
```python

- start_index和end_index全为正（+）索引的情况
```python
>>>a[1:6]
>>> [1, 2, 3, 4, 5]
#step=1，从左往右取值，start_index=1到end_index=6同样表示从左往右取值。

>>>a[1:6:-1]
>>> []
#输出为空列表，说明没取到数据。
#step=-1，决定了从右往左取值，而start_index=1到end_index=6决定了从左往右取值，两者矛盾，所以为空。

>>>a[6:1]
>>> []
#同样输出为空列表。
#step=1，决定了从左往右取值，而start_index=6到end_index=1决定了从右往左取值，两者矛盾，所以为空。

>>>a[:6]
>>> [0, 1, 2, 3, 4, 5]
#step=1，从左往右取值，从“起点”开始一直取到end_index=6。

>>>a[:6:-1]
>>> [9, 8, 7]
$step=-1，从右往左取值，从“终点”开始一直取到end_index=6。

>>>a[6:]
>>> [6, 7, 8, 9]
#step=1，从左往右取值，从start_index=6开始，一直取到“终点”。

>>>a[6::-1]
>>> [6, 5, 4, 3, 2, 1, 0]
#step=-1，从右往左取值，从start_index=6开始，一直取到“起点”。
```

- start_index和end_index全为负（-）索引的情况
```python
>>>a[-1:-6]
>>> []
#step=1，从左往右取值，而start_index=-1到end_index=-6决定了从右往左取值，两者矛盾，所以为空。
#索引-1在-6的右边（如上图）

>>>a[-1:-6:-1]
>>> [9, 8, 7, 6, 5]
#step=-1，从右往左取值，start_index=-1到end_index=-6同样是从右往左取值。
#索引-1在6的右边（如上图）

>>>a[-6:-1]
>>> [4, 5, 6, 7, 8]
#step=1，从左往右取值，而start_index=-6到end_index=-1同样是从左往右取值。
#索引-6在-1的左边（如上图）

>>>a[:-6]
>>> [0, 1, 2, 3]
#step=1，从左往右取值，从“起点”开始一直取到end_index=-6。

>>>a[:-6:-1]
>>> [9, 8, 7, 6, 5]
#step=-1，从右往左取值，从“终点”开始一直取到end_index=-6。

>>>a[-6:]
>>> [4, 5, 6, 7, 8, 9]
#step=1，从左往右取值，从start_index=-6开始，一直取到“终点”。

>>>a[-6::-1]
>>> [4, 3, 2, 1, 0]
#step=-1，从右往左取值，从start_index=-6开始，一直取到“起点”。
```
- start_index和end_index正（+）负（-）混合索引的情况
```python
>>>a[1:-6]
>>> [1, 2, 3]
#start_index=1在end_index=-6的左边，因此从左往右取值，而step=1同样决定了从左往右取值，因此结果正确

>>>a[1:-6:-1]
>>> []
#start_index=1在end_index=-6的左边，因此从左往右取值，但step=-则决定了从右往左取值，两者矛盾，因此为空。

>>>a[-1:6]
>>> []
#start_index=-1在end_index=6的右边，因此从右往左取值，但step=1则决定了从左往右取值，两者矛盾，因此为空。

>>>a[-1:6:-1]
>>> [9, 8, 7]
#start_index=-1在end_index=6的右边，因此从右往左取值，而step=-1同样决定了从右往左取值，因此结果正确。
```
- 连续切片操作
```python
>>>a[:8][2:5][-1:]
>>> [4]
#相当于：
a[:8]=[0, 1, 2, 3, 4, 5, 6, 7]
a[:8][2:5]= [2, 3, 4]
a[:8][2:5][-1:] = 4
#理论上可无限次连续切片操作，只要上一次返回的依然是非空可切片对象。
```
- 切片操作的三个参数可以用表达式
```python
>>>a[2+1:3*2:7%3]
>>> [3, 4, 5]
#即：a[2+1:3*2:7%3] = a[3:6:1]
```
- 其他对象的切片操作
前面的切片操作说明都以list为例进行说明，但实际上可进行的切片操作的数据类型还有很多，包括元组、字符串等等。
```python
>>> (0, 1, 2, 3, 4, 5)[:3]
>>> (0, 1, 2)

#元组的切片操作

>>>'ABCDEFG'[::2]
>>>'ACEG'
#字符串的切片操作

>>>for i in range(1,100)[2::3][-10:]: 
       print(i)
```	   
就是利用range函数生成1-99的整数，然后取3的倍数，再取最后十个。

- 常用切片操作
以列表：`a = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]` 为说明对象
```python
#1.取偶数位置
>>>b = a[::2]
[0, 2, 4, 6, 8]

#2.取奇数位置
>>>b = a[1::2]
[1, 3, 5, 7, 9]

#3.拷贝整个对象
>>>b = a[:] #★★★★★
>>>print(b) #[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>>print(id(a)) #41946376
>>>print(id(b)) #41921864
#或
>>>b = a.copy()
>>>print(b) #[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
>>>print(id(a)) #39783752
>>>print(id(b)) #39759176

#需要注意的是：[:]和.copy()都属于“浅拷贝”，只拷贝最外层元素，内层嵌套元素则通过引用，而不是独立分配内存。
>>>a = [1,2,['A','B']]
>>>print('a={}'.format(a))
>>>b = a[:]
>>>b[0] = 9 #修改b的最外层元素，将1变成9
>>>b[2][0] = 'D' #修改b的内嵌层元素
>>>print('a={}'.format(a))
>>>print('b={}'.format(b))
>>>print('id(a)={}'.format(id(a)))
>>>print('id(b)={}'.format(id(b)))
a=[1, 2, ['A', 'B']] #原始a
a=[1, 2, ['D', 'B']] #b修改内部元素A为D后，a中的A也变成了D，说明共享内部嵌套元素，但外部元素1没变。
b=[9, 2, ['D', 'B']] #修改后的b
id(a)=38669128
id(b)=38669192

#4.修改单个元素
>>>a[3] = ['A','B']
[0, 1, 2, ['A', 'B'], 4, 5, 6, 7, 8, 9]

#5.在某个位置插入元素
>>>a[3:3] = ['A','B','C']
[0, 1, 2, 'A', 'B', 'C', 3, 4, 5, 6, 7, 8, 9]
>>>a[0:0] = ['A','B']
['A', 'B', 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]

#6.替换一部分元素
>>>a[3:6] = ['A','B']
[0, 1, 2, 'A', 'B', 6, 7, 8, 9]
```
- 总结    
1、`start_index、end_index、step`可同为正、同为负，也可正负混合使用。但必须遵循一个原则，否则无法正确切取到数据：
   当`start_index`的位置在`end_index`的左边时，表示从左往右取值，此时`step`必须是正数（同样表示从左往右）；
   当`start_index`的位置在`end_index`的右边时，表示从右往左取值，此时`step`必须是负数（同样表示从右往左），
   即两者的取值顺序必须是相同的。对于特殊情况，当`start_index`或`end_index`省略时，起始索引和终止索引由`step`的正负来决定，
   不会存在取值方向出现矛盾的情况（即不会返回空列表[]），但正和负取到的结果是完全不同的，因为一个向左一个向右。  
2、在利用切片时，`step`的正负是必须要考虑的，尤其是当`step`省略时。
   比如`a[-1:]`，很容易就误认为是从“终点”开始一直取到“起点”，即`a[-1:]= [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]`，
   但实际上`a[-1:]=a[-1]=9`，原因在于step=1表示从左往右取值，而起始索引`start_index=-1`本身就是对象的最右边的元素了，再往右已经没数据了，因此只有`a[-1]`一个元素。


#### 列表主要方法
```python
>>> fruits = ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
```
- list.append(x)
在列表的末尾添加一个元素。相当于`a[len(a):] = [x]`
```python
>>>fruits.append(['grape', 'Strawberry'])
>>>fruits
['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana', ['grape', 'Strawberry']]
```

- list.extend(iterable)
使用可迭代对象中的所有元素来扩展列表。相当于`a[len(a):] = iterable` 
```python
>>>fruits.extend(['grape', 'Strawberry'])
>>>fruits
['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana', 'grape', 'Strawberry']
```

- list.insert(i, x)
在给定的位置插入一个元素。第一个参数是要插入的元素的索引，所以`a.insert(0, x)`插入列表头部， `a.insert(len(a), x)` 等同于`a.append(x)`
```python
>>>fruits.insert(1, 'grape')
>>>fruits
['orange', 'grape', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
```

- list.remove(x)
移除列表中第一个值为`x`的元素。如果没有这样的元素，则抛出`ValueError`异常
```python
>>>fruits.remove('apple')
>>>fruits
['orange', 'pear', 'banana', 'kiwi', 'apple', 'banana']
```

- list.pop([i])
删除列表中给定位置的元素并返回它。如果没有给定位置，`a.pop()`将会删除并返回列表中的最后一个元素。
（ 方法签名中 `i` 两边的方括号表示这个参数是可选的，而不是要你输入方括号。你会在 Python 参考库中经常看到这种表示方法)。
```python
>>> fruits.pop()
'pear'
```
- list.clear()
删除列表中所有的元素。相当于 `del a[:]` 
```python
>>>fruits.clear()
>>>fruits
[]
```

- list.index(x[, start[, end]])
返回列表中第一个值为`x`的元素的从零开始的索引。如果没有这样的元素将会抛出`ValueError`异常
```python
>>> fruits.index('banana')
3
>>> fruits.index('banana', 4)  # Find next banana starting a position 4
6
```

可选参数`start` 和 `end` 是切片符号，用于将搜索限制为列表的特定子序列。返回的索引是相对于整个序列的开始计算的，而不是`start`参数

- list.count(x)
返回元素`x`在列表中出现的次数
```python
>>> fruits.count('apple')
2
>>> fruits.count('tangerine')
0
```

- list.sort(key=None, reverse=False)
对列表中的元素进行排序（参数可用于自定义排序，解释请参见`sorted()`）
```python
>>>fruits.sort()
>>>fruits
['apple', 'apple', 'banana', 'banana', 'kiwi', 'orange', 'pear']
```

- list.reverse()
反转列表中的元素
```python
>>> fruits.reverse()
>>> fruits
['banana', 'apple', 'kiwi', 'banana', 'pear', 'apple', 'orange']
```

- list.copy()
返回列表的一个浅拷贝。相当于`a[:]` 
```python

```
你可能已经注意到，像`insert ，remove` 或者 `sort` 方法，只修改列表，没有打印出返回值——它们返回默认值 `None` 。这是Python中所有可变数据结构的设计原则。
#### 列表推导式
列表推导式提供了一个更简单的创建列表的方法。常见的用法是把某种操作应用于序列或可迭代对象的每个元素上，然后使用其结果来创建列表，或者通过满足某些特定条件元素来创建子序列
```python
squares = list(map(lambda x: x**2, range(10)))
##或者，等价于

squares = [x**2 for x in range(10)]

>>> [(x, y) for x in [1,2,3] for y in [3,1,4] if x != y]
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
#而它等价于

>>>
>>> combs = []
>>> for x in [1,2,3]:
...     for y in [3,1,4]:
...         if x != y:
...             combs.append((x, y))
...
>>> combs
[(1, 3), (1, 4), (2, 3), (2, 1), (2, 4), (3, 1), (3, 4)]
#注意在上面两个代码片段中， for 和 if 的顺序是相同的。
```
#### 列表遍历
```python
list=['1','2','3']
for value in list: #末尾加上冒号
    print(value) #每次循环都把list列表中的值赋给value,赋值从索引号0开始#循环的语句需要缩进
 
#结果：
1
2
3
 
 
list=['1','2','3','4','5','6','7']
for value in list[3:]: #遍历索引3之后的数值
    print(value)
 
#结果：
4
5
6
7
```
#### 列表操作符
- in、not in    
判断元素是否在列表中
```python
>>>my_list = [2, 4, 6]
>>>2 in my_list
True
```
- 加号(+)
```python
>>>x = [2,3,4]
>>>y = [7,8,9]
>>>z = x + y
>>>z
[2, 3, 4, 7, 8, 9]
```
- 星号(*)
```python
>>>my = [2,3,4]
>>>x = my * 2
>>>x
[2, 3, 4, 2, 3, 4]
```
#### 列表、元组、字典互转
- 字典
```python
dict = {'name': 'Zara', 'age': 7}
#字典转为字符串
str(dict)
#字典可以转为元组
print(tuple(dict))
#字典可以转为元组
print(tuple(dict.values()))
#字典转为列表
print(list(dict))
#字典转为列表
print(list(dict.values()))
```
- 列表
```python
nums=[1, 3, 5, 7, 9, 11, 13];
#列表转为字符串
str(nums)
#列表转为元组
tuple(nums)
#列表不可以转为字典
```
- 元组
```python
tup=(1, 2, 3, 4, 5,6,7,8)
#元组转为字符串
tup.__str__()
#元组转为列表
list(tup)
#元组不可以转为字典
```
- 字符串
```python
str="(1,2,3)"
#字符串转为元组
tuple(eval(str))
#字符串转为列表
list(eval("(1,2,3)"))
#字符串转为字典
str1="{'name':'ljq', 'age':24}"
eval(str1)
```
#### 引用
- [彻底搞懂Python切片操作](https://www.jianshu.com/p/15715d6f4dad)
- [python官方list文档](https://docs.python.org/zh-cn/3/tutorial/datastructures.html#data-structures)
- [Python魔法方法指南](https://pyzh.readthedocs.io/en/latest/python-magic-methods-guide.html)
