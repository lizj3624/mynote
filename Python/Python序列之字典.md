### 字典
- 字典的键值不允许对应多值
- 字典的键值是可哈希的
#### 字典创建
```python
#常规创建
aDict = {'name':'Jack Lee', 'age': 30, 'gender': 'man'}

#动态创建
aDict = {}
aDict['name'] = 'Jack Lee'
aDict['age'] = 30
aDict['gender'] = 'man'

#dict函数创建
aDict = dict('name'='Jack Lee', 'age'= 30, 'gender'='man')

aDict = dict([('name', 'Jack Lee'), ('age', 30), ('gender', 'man')])

aDict = dict.fromkeys(['name', 'age', 'gender'])
print(aDict)
{'name': None, 'age': None, 'gender': None}
```
#### 字典访问
```python
aDict = dict('name'='Jack Lee', 'age'= 30, 'gender'='man')
print(aDict['name'])
Jack Lee

print(aDict.get('name'))  #dict.get(key, default), 支持设置默认值
Jack Lee
```
#### 字典更新
```python
aDict = dict('name'='Jack Lee', 'age'= 30, 'gender'='man')
aDict['name'] = 'Jack Song'
print(aDict['name'])
Jack Song

#update()
aDict = dict('name'='Jack Lee', 'age'= 30, 'gender'='man')
bDict = {'name':'Jack Song', 'phone':'13111111144'}
aDict.update(bDict)
print(aDict)
{'name': 'Jack Song', 'age': 30, 'gender': 'man', 'phone': '13111111144'}
```
#### 字典遍历
```python
aDict = {'name': 'Jack Song', 'age': 30, 'gender': 'man', 'phone': '13111111144'}
#keys()  函数以列表返回一个字典所有的键
print(aDict.keys())
['name', 'age', 'gender', 'phone']
for key in aDict.keys():
    ...

#values() 函数以列表返回字典中的所有值
print(aDict.values())
['Jack Song', 30, 'man', '13111111144']
for value in aDict.Vaules():
    ...

#items() 函数以列表返回可遍历的(键, 值) 元组数组
print(aDict.items())
[('name', 'Jack Song'), ('age', 30), ('gender', 'man'), ('phone', '13111111144')]
for key, value in aDict.items():
    ...
```
#### 字典判断
```python
#has_key()方法在python3不存在，用如下替代
bDict = {'name':'Jack Song', 'phone':'13111111144'}
'name' in bDict.keys()
True
```
#### 引用
- [Python官方dict](https://docs.python.org/zh-cn/3/library/stdtypes.html#dict)