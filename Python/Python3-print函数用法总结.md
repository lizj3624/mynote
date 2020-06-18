### Python3 print函数用法总结
Python3的print是一个函数，必须有`()`
```python
print('Hello')
```
#### 1、输出字符串、数字、序列
```
>>> print('Hello World!!!')
Hello World!!!
>>> print(100)
100
>>> my_list = [2, 4, 'lee']
>>> print(my_list)
[2, 4, 'lee']
>>> my_list = (3, 4, 'lee')
>>> print(my_list)
(3, 4, 'lee')
>>> my_dict = {'my_name':'lee', 'age': 23}
>>> print(my_dict)
{'my_name': 'lee', 'age': 23}
```

#### 2、格式化输出
- 格式化输出整数、浮点数、字符串
```python
>>> my_str = 'Hello World'
>>> print('str: %s, len: %d'%(my_str, len(my_str)))
str: Hello World, len: 11

>>> pi = 3.141592653
>>> print('pi: %10.3f'%(pi))
pi:      3.142
>>> print('pi: %.3f'%(pi))
pi: 3.142
```

- 格式化输出八进制、十六进制
```python
#%x — hex 十六进制
#%d — dec 十进制
#%o — oct 八进制

>>> nHex = 0xFF
>>> print('nHex = %x, nDec = %d, nOct = %o'%(nHex, nHex, nHex))
nHex = ff, nDec = 255, nOct = 377
```

- 格式化输出字典
```python
>>> print('%(language)s has %(number)03d quote types.' %
...       {'language': "Python", "number": 2})
Python has 002 quote types.
```

#### 3、自动换行
`print`会自动在行末加上回车, 如果不需回车，只需在`print`语句的结尾添加一个逗号`,` 就可以改变它的行为。
```python
>>> for i in range(0,3):
...     print(i)
...
0
1
2

>>> for i in range(0,3):
...     print(i, end = '')
...
012>>>
```

#### 4、引用
- [官方print用法](https://docs.python.org/zh-cn/3/library/stdtypes.html#printf-style-string-formatting)
