### 集合set
- 集合(set)是一组无序排列的可哈希的值
- 由于集合是无序的，因此不能为集合创建索引或者执行切片操作，也不可以通过键值获取集合中的元素的值
- 集合(set)分为两种类型：    
  1）可变集合set，可以添加、删除，是不可哈希，不可以做字典的键值，也不可做其他集合中的元素   
  2）不可变集合fronzenset，不可添加、删除，可哈希
- 集合的主要作用如下    
  1）去重，把一个列表变成集合，就自动去重了    
  2）关系测试，测试两组数据之前的交集、差集、并集等关系   
#### 集合创建
```
# 创建可变集合
>>> my_set = set('abcdec')
>>> print(my_set)
{'b', 'd', 'a', 'c', 'e'}

#创建不可变集合
>>> my_frozenset = frozenset('abcdec')
>>> print(my_frozenset)
frozenset({'b', 'd', 'a', 'c', 'e'})
>>> type(my_frozenset)
<class 'frozenset'>

```
#### 集合访问
```python
>>> 'a' in my_set
True
>>> 'c' in my_set
True
>>> for i in my_set:
...     print(i)
...
b
d
a
c
e
```
#### 集合更新 
```python
#只适应于可变集合
>>> my_set.add('f')
>>> print(my_set)
{'b', 'd', 'f', 'a', 'c', 'e'}
>>> print(my_set)
{'b', 'd', 'f', 'a', 'c', 'e'}
>>> my_set.update('egh')
>>> print(my_set)
{'g', 'b', 'd', 'h', 'f', 'a', 'c', 'e'}

>>> print(my_set)
{'g', 'b', 'd', 'h', 'f', 'a', 'c', 'e'}
# remove 删除集合中元素，元素不存在时有错误提示
>>> my_set.remove('a')
>>> my_set.remove('j')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'j'

# discard 删除集合中元素，元素不存在时没有提示
>>> my_set.discard('j')
>>> my_set.discard('b')

# 删除集合
del my_set
```
#### 集合操作
```python
#两个集合相等，跟顺序无关，只跟元素有关
>>> set('shop') == set('posh')
True
>>> set('shop') == set('poshe')
False

#子集和超集
>>> set('shop') < set('poshe')
True

#联合 |
>>> my_set = set('abcdef')
>>> your_set = set('efghij')
>>> u = my_set | your_set
>>> print(u)
{'g', 'i', 'b', 'd', 'h', 'f', 'a', 'c', 'j', 'e'}

# 交集 &
>>> u = my_set & your_set
>>> print(u)
{'f', 'e'}

# 差集 -
>>> u = my_set - your_set
>>> print(u)
{'c', 'a', 'b', 'd'}

# 对称差分 ^
>>> u = my_set ^ your_set
>>> print(u)
{'g', 'i', 'a', 'c', 'b', 'd', 'h', 'j'}


#union update |=
>>> s = set('cheeseshop')
>>> u = frozenset(s)
>>> s |= set('pypi')
>>> print(s)
{'i', 'h', 'p', 's', 'o', 'c', 'e', 'y'}

# 交集 更新 &=
>>> s = set(u)
>>> s &= set('shop')
>>> print(s)
{'s', 'h', 'p', 'o'}

# 差集 更新 -=
>>> s = set(u)
>>> s -= set('shop')
>>> print(s)
{'c', 'e'}

# 对称差分 更新 ^=
>>> s = set(u)
>>> t = frozenset('bookshop')
>>> s ^= t
>>> print(s)
{'e', 'k', 'b', 'c'}
```
#### 集合内建方法
```python
#标准函数
>>> my_set = set('abcd')
>>> len(my_set)
4

# 集合内建方法
>>> my_set = set('abcd')
>>> y_set = set('cded')
>>> my_set.issubset(y_set)
False
>>> my_set.issuperset(y_set)
False
>>> t = my_set.intersection(y_set)
>>> print(t)
{'c', 'd'}
>>> t = my_set.difference(y_set)
>>> print(t)
{'a', 'b'}
>>> t = my_set.symmetric_difference(y_set)
>>> print(t)
{'e', 'a', 'b'}
```
- 详细可以看官方文档
#### 引用
- [官方set文档](https://docs.python.org/zh-cn/3/library/stdtypes.html?highlight=set#set)