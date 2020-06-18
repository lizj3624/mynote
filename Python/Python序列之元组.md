### 元组
#### 元组创建
```python
aTuple = ('123', 'abc', 45)  #元组用括号
bTuple = ('c',) #只有一个元素的元组必须用逗号(,)，不然就当做是字符串

>>> cTuple = tuple('str') #将字符串转换成元组
('s', 't', 'r')
```
#### 元组访问
```python
aTuple = ('123', 'abc', 45, 4.6)
#通过下标访问
print(aTuple[2])

#通过切片访问
print(aTuple[1:3])
```
#### 元组更新
```python
# 元组是不可改变的类型，不能更新、改变、删除元组的元素，但是可以通过现有元组创建新的元组
bTuple = aTuple[1:3]
print(bTuple)
```
#### 元组操作符
```python
# + 号
aTuple = ('123', 'abc', 45, 4.6)
bTuple = ('ef', 87)
print(aTuple + bTuple)
('123', 'abc', 45, 4.6, 'ef', 87)

# * 号
aTuple = ('123', 'abc', 45, 4.6)
print(aTuple * 2)
('123', 'abc', 45, 4.6, '123', 'abc', 45, 4.6)

# in、 not in
aTuple = ('123', 'abc', 45, 4.6)
'123' in aTuple
```
#### 内建函数
```python
aTuple = ('123', 'abc', 45, 4.6)
str(aTuple)
mix(aTuple)
max(aTuple)
len(aTuple)

#将元组转换列表
list(aTuple)
#将列表转成成元组
tuple(alist)  #alist是一个列表

#元组因为不可变，因此没有列表的排序、替换、添加、删除等操作
```
#### 引用