### 内置函数
- len()    
返回对象的长度（元素个数）。实参可以是序列（如 string、bytes、tuple、list 或 range 等）或集合（如 dictionary、set 或 frozen set 等）。

- min()    
返回可迭代对象中最小的元素，或者返回两个及以上实参中最小的。

- max()    
返回可迭代对象中最大的元素，或者返回两个及以上实参中最大的。

- type()   
传入一个参数时，返回 `object` 的类型。 返回值是一个 `type` 对象，通常与 `object.__class__` 所返回的对象相同。
```python
>>> class X:
...     a = 1
...
>>> X = type('X', (object,), dict(a=1))
```

- zip()   
创建一个聚合了来自每个可迭代对象中的元素的迭代器。
返回一个元组的迭代器，其中的第 i 个元组包含来自每个参数序列或可迭代对象的第 i 个元素。 当所输入可迭代对象中最短的一个被耗尽时，迭代器将停止迭代。 当只有一个可迭代对象参数时，它将返回一个单元组的迭代器。 不带参数时，它将返回一个空迭代器。
```python
>>> x = [1, 2, 3]
>>> y = [4, 5, 6]
>>> zipped = zip(x, y)
>>> list(zipped)
[(1, 4), (2, 5), (3, 6)]
>>> x2, y2 = zip(*zip(x, y))
>>> x == list(x2) and y == list(y2)
True
```
- [详细参看一下官方文档](https://docs.python.org/zh-cn/3/library/functions.html)