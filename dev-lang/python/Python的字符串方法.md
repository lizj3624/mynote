### 字符串的创建
- 字符串是不可变类型，不能修改字符串中的某个字符，可以通过拼接字符串，组成新的字符串
- 将一串字符可通过单引号`''`, `""`, `""" """`括起来组成字符串
```python
aString = 'Hello World'
bString = "Hello World"
```
#### 字符串操作符
- 原始字符串操作符(r/R)
```python
>>> print(r'\n')
\n

>>> print('\n')

```
- UniouCode 字符串操作符(u/U)
```python
u'abc'
```
### 字符串的方法

#### 1、字符串大小写转换的方法
   - str.capitalize()    
     将首字母转换成大写，需要注意的是如果首字没有大写形式，则返回原字符串
     ```python
     >>> my_str = str('aabcDefa')
     >>> my_str.capitalize()
     'Aabcdefa'
     ```
   - str.casefold()   
   将字符串转换成小写，Unicode 编码中凡是有对应的小写形式的，都会转换。
     ```python
     >>> my_str = str('aabcDefa')
     >>> my_str.casefold()
     'aabcdefa'
     ```
   - str.lower()    
   将字符串转换成小写，其仅对 ASCII 编码的字母有效
     ```python
     >>> my_str = str('aabcDefa')
     >>> my_str.lower()
     'aabcdefa'
     ```
   - str.swapcase()    
   对字符串字母的大小写进行反转
     ```python
     >>> my_str = str('aabcDefa')
     >>> my_str.swapcase()
     'AABCdEFA'
     ```
     但需要注意的是`s.swapcase().swapcase() == s` 不一定为真
   - str.title()    
   将字符串中每个“单词”首字母大写。其判断“单词”的依据则是基于空格和标点，所以应对英文撇好所有格或一些英文大写的简写时，会出错
     ```python
     >>> my_str = 'hello world!!!'
     >>> my_str.title()
     'Hello World!!!'
     ```
   - str.upper()    
   将字符串所有字母变为大写，会自动忽略不可转成大写的字符    
   ```python
    >>> my_str = str('aabcDefa')
    >>> my_str.upper()
    'HELLO WORLD!!!'
   ```
   需要注意的是`s.upper().isupper()` 不一定为 `True`
   
#### 2、字符串格式输出的方法
   - str.center(width[, fillchar])    
   将字符串按照给定的宽度居中显示，可以给定特定的字符填充多余的长度，如果指定的长度小于字符串长度，则返回原字符串。
   ```python
   >>> my_str = 'abcdef'
   >>> my_str.center(10, '*')
   '**abcdef**'
   ```
   - str.expandtabs(tabsize=8)    
   用指定的空格替代横向制表符，使得相邻字符串之间的间距保持在指定的空格数以内。
   ```python
   >>> '01\t012\t0123\t01234'.expandtabs()
   '01      012     0123    01234'
   >>> '01\t012\t0123\t01234'.expandtabs(4)
   '01  012 0123    01234'
   ```
   - str.format(*args, **kwargs)    
   格式化字符串的语法比较繁多，官方文档已经有比较详细的[examples](https://docs.python.org/zh-cn/3/library/string.html#format-examples)

   - str.format_map(mapping)    
   类似`str.format(*args, **kwargs)`，不同的是`mapping`是一个字典对象
   ```python
   >>> People = {'name':'john', 'age':56}
   >>> 'My name is {name},i am {age} old'.format_map(People)
   # 'My name is john,i am 56 old'
   ```
   - str.ljust(width[, fillchar])    
   返回指定长度的字符串，字符串内容居左如果长度小于字符串长度，则返回原始字符串，默认填充为 ASCII 空格，可指定填充的字符串
   ```python
   >>> my_str = 'abcdef'
   >>> my_str.ljust(10, '*')
   'abcdef****'
   ```
   - str.rjust(width[, fillchar])
   返回指定长度的字符串，字符串内容居右如果长度小于字符串长度，则返回原始字符串，默认填充为 ASCII 空格，可指定填充的字符串
   ```python
   
   >>> my_str = 'abcdef'
   >>> my_str.rjust(10, '*')
   '****abcdef'
   ```
   - str.zfill(width)    
   用 `0` 填充字符串，并返回指定宽度的字符串
   ```python
   >>> my_str = 'abcdef'
   >>> my_str.zfill(10)
   '0000abcdef'
   >>> my_str = '-123'
   >>> my_str.zfill(10)
   '-000000123'
   ```

#### 3、字符串搜索定位与替换的方法
   - str.count(sub[, start[, end]])    
   反回子字符串 sub 在 [start, end] 范围内非重叠出现的次数
   ```python
   >>> my_str = 'abcdef'
   >>> my_str.count('bc')
   1
   >>> my_str.count('bc', 1, 5)
   1
   ```
   - str.find(sub[, start[, end]])    
   返回子字符串`sub`在`s[start:end]`切片内被找到的最小索引。 可选参数`start`与`end`会被解读为切片表示法。 如果`sub` 未被找到则返回`-1`
   ```python
   >>> my_str = 'abcdef'
   >>> my_str.find('bc', 1, 5)
   1
   ```
   - str.index(sub[, start[, end]])   
   与`find() rfind()` 类似，不同的是如果找不到，就会引发`ValueError`
   ```python
   >>> my_str = 'abcdef'
   >>> my_str.index('bc', 1, 5)
   1
   ```
   - str.rfind(sub[, start[, end]])   
   与`find`类似，最右的索引

   - str.rindex(sub[, start[, end]])    
   与`index`类似，最右的索引

   - str.strip([chars])    
   返回原字符串的副本，移除其中的前导和末尾字符。 `chars`参数为指定要移除字符的字符串。 如果省略或为`None`，则`chars` 参数默认移除空格符
   ```python
   >>> my_str = '   abcdef   '
   >>> my_str.strip()
   'abcdef'
   ```
   - str.lstrip([chars])   
   与`strip`类似，移除左侧的`chars`或者空格
   ```python
   >>> my_str = '   abcdef   '
   >>> my_str.lstrip()
   'abcdef   '
   ```
   - str.rstrip([chars])    
   与`strip`类似，移除右侧的`chars`或者空格
   ```python
   >>> my_str = '   abcdef   '
   >>> my_str.rstrip()
   '   abcdef'
   ```
   - str.replace(old, new[, count])    
   返回字符串的副本，其中出现的所有子字符串 old 都将被替换为 new
   ```python
   >>> my_str = 'abcdef'
   >>> my_str.replace('bc', 'www')
   'awwwdef'
   ```
   - str.translate(table)    
   
   - static str.maketrans(x[, y[, z]])     
   `maktrans`是一个静态方法，用于生成一个对照表，以供`translate`使用。
如果`maktrans`仅一个参数，则该参数必须是一个字典，字典的`key`要么是一个`Unicode`编码（一个整数），要么是一个长度为 `1` 的字符串，字典的 `value` 则可以是任意字符串、`None`或者 `Unicode` 编码。
   ```python
   a = 'dobi'
   ord('o')
   # 111

   ord('a')
   # 97

   hex(ord('狗'))
   # '0x72d7'

   b = {'d':'dobi', 111:' is ', 'b':97, 'i':'\u72d7\u72d7'}
   table = str.maketrans(b)

   a.translate(table)
   # 'dobi is a狗狗'
   ```

#### 4、字符串的联合与分割的方法
   - str.partition(sep)   
   在 sep 首次出现的位置拆分字符串，返回一个 3 元组，其中包含分隔符之前的部分、分隔符本身，以及分隔符之后的部分。 如果分隔符未找到，则返回的 3 元组中包含字符本身以及两个空字符串。
   ```python
   >>> 'dog wow wow jiao'.partition('wow')
   ('dog ', 'wow', ' wow jiao')

   >>> 'dog wow wow jiao'.partition('dog')
   # ('', 'dog', ' wow wow jiao')

   >>> 'dog wow wow jiao'.partition('jiao')
   ('dog wow wow ', 'jiao', '')

   >>> 'dog wow wow jiao'.partition('ww')
   ('dog wow wow jiao', '', '')
   ```
   - str.rpartition(sep)    
   与`partition`类似, 从右边`sep`切分
   ```python
   >>> 'dog wow wow jiao'.rpartition('wow')
   ('dog wow ', 'wow', ' jiao')

   >>> 'dog wow wow jiao'.rpartition('dog')
   ('', 'dog', ' wow wow jiao')

   >>> 'dog wow wow jiao'.rpartition('jiao')
   ('dog wow wow ', 'jiao', '')

   >>> 'dog wow wow jiao'.rpartition('ww')
   ('', '', 'dog wow wow jiao')
   ```
   - str.split(sep=None, maxsplit=-1)    
   返回一个由字符串内单词组成的列表，使用`sep`作为分隔字符串
   ```python
   >>> '1,2,3'.split(',')
   ['1', '2', '3']
   >>> '1,2,3'.split(',', maxsplit=1)
   ['1', '2,3']
   >>> '1,2,,3,'.split(',')
   ['1', '2', '', '3', '']
   >>> '1 2 3'.split()
   ['1', '2', '3']
   >>> '1 2 3'.split(maxsplit=1)
   ['1', '2 3']
   >>> '   1   2   3   '.split()
   ['1', '2', '3']
   ```
   - str.rsplit(sep=None, maxsplit=-1)    
   与`split`类似，返回一个由字符串内单词组成的列表，使用`sep`作为分隔字符串。 如果给出了`maxsplit`，则最多进行`maxsplit` 次拆分，从`最右边 `开始

   - str.splitlines([keepends])    
   字符串以行界符为分隔符拆分为列表；当 keepends 为True，拆分后保留行界符，能被识别的行界符见
   ```python
   >>> 'ab c\n\nde fg\rkl\r\n'.splitlines()
   ['ab c', '', 'de fg', 'kl']
   >>> 'ab c\n\nde fg\rkl\r\n'.splitlines(keepends=True)
   ['ab c\n', '\n', 'de fg\r', 'kl\r\n']

   >>> "".splitlines()， ''.split('\n')      #注意两者的区别
   ([], [''])
   >>> "One line\n".splitlines()
   (['One line'], ['Two lines', ''])
   ```
   - str.join(iterable)    
   用指定的字符串，连接元素为字符串的可迭代对象
   ```python
   >>> '-'.join(['2012', '3', '12'])
   '2012-3-12'

   >>> '-'.join([2012, 3, 12])
   TypeError: sequence item 0: expected str instance, int found

   '-'.join(['2012', '3', b'12'])  #bytes 为非字符串
   >>> TypeError: sequence item 2: expected str instance, bytes found

   >>> '-'.join(['2012'])  
   '2012'

   >>> '-'.join([])
   ''

   >>> '-'.join([None])
   TypeError: sequence item 0: expected str instance, NoneType found

   >>> '-'.join([''])
   ''

   >>> ','.join({'dobi':'dog', 'polly':'bird'})
   'dobi,polly'
   ```

#### 5、字符串条件判断的方法
   - str.endswith(suffix[, start[, end]])    
   如果字符串以指定的`suffix`结束返回`True`，否则返回`False`。 `suffix`也可以为由多个供查找的后缀构成的元组
   ```python
   >>> text = 'outer protective covering'
   >>> text.endswith('ing')
   True

   >>> text.endswith(('gin', 'ing'))
   True
   
   >>> text.endswith('ter', 2, 5)
   True

   >>> text.endswith('ter', 2, 4)
   False
   ```
   - str.startswith(prefix[, start[, end]])   
   如果字符串以指定的`prefix`开始则返回`True`，否则返回`False`。 `prefix`也可以为由多个供查找的前缀构成的元组
   ```python
   >>> text = 'outer protective covering'
   >>> text.startswith('outer')
   True
   ```
   - str.isalnum()   
   字符串和数字的任意组合，即为真，简而言之：只要 `c.isalpha(), c.isdecimal(), c.isdigit(), c.isnumeric()` 中任意一个为真，则`c.isalnum()`为真。
   ```python
   'dobi'.isalnum()
   # True

   'dobi123'.isalnum()
   # True

   '123'.isalnum()
   # True

   '徐'.isalnum()
   # True

   'dobi_123'.isalnum()
   # False

   'dobi 123'.isalnum()
   # False

   '%'.isalnum()
   # False
   ```
   - str.isalpha()   
   `Unicode`字符数据库中作为 `“Letter”`（这些字符一般具有 `“Lm”, “Lt”, “Lu”, “Ll”, or “Lo”` 等标识，不同于` Alphabetic`） 的，均为真。
   ```python
   'dobi'.isalpha()
   # True

   'do bi'.isalpha()
   # False

   'dobi123'.isalpha()
   # False

   '徐'.isalpha()
   # True
   ```
   - str.isascii()    
   如果字符串为空或所有字符均为`ASCII`字符则返回真值，否则返回假值。 `ASCII`字符的码位范围为`U+0000-U+007F`。

   - str.isdecimal()
   - str.isdigit()
   - str.isnumeric()    
   三个方法的区别在于对 Unicode 通用标识的真值判断范围不同：
   `isdecimal: Nd,`
   `isdigit: No, Nd,`
   `isnumeric: No, Nd, Nl`
   `digit` 与 `decimal` 的区别在于有些数值字符串，是`digit` 却非 `decimal`
   ```python
   num = '\u2155'
   print(num)
   # ⅕
   num.isdecimal(), num.isdigit(), num.isnumeric()
   # (False, False, True)

   num = '\u00B2'
   print(num)
   # ²
   num.isdecimal(), num.isdigit(), num.isnumeric()
   # (False, True, True)

   num = "1"  #unicode
   num.isdecimal(), num.isdigit(), num.isnumeric()
   # (Ture, True, True)

   num = "'Ⅶ'" 
   num.isdecimal(), num.isdigit(), num.isnumeric()
   # (False, False, True)

   num = "十"
   num.isdecimal(), num.isdigit(), num.isnumeric()
   # (False, False, True)

   num = b"1" # byte
   num.isdigit()   # True
   num.isdecimal() # AttributeError 'bytes' object has no attribute 'isdecimal'
   num.isnumeric() # AttributeError 'bytes' object has no attribute 'isnumeric'
   ```
   - str.isidentifier()    
   判断字符串是否可为合法的标识符
   ```python
   'def'.isidentifier()
   # True

   'with'.isidentifier()
   # True

   'false'.isidentifier()
   # True

   'dobi_123'.isidentifier()
   # True

   'dobi 123'.isidentifier()
   # False

   '123'.isidentifier()
   # False
   ```
   - str.islower()   
   ```python
   '徐'.islower()
   # False

   'ß'.islower()   #德语大写字母
   # False

   'a徐'.islower()
   # True

   'ss'.islower()
   # True

   '23'.islower()
   # False

   'Ab'.islower()
   # False
   ```
   - str.isprintable()   
   判断字符串的所有字符都是可打印字符或字符串为空。`Unicode`字符集中`“Other” “Separator”` 类别的字符为不可打印的字符（但不包括 `ASCII` 的空格（`0x20`））。
   ```python
   'dobi123'.isprintable()
   # True

   'dobi123\n'.isprintable()
    False

   'dobi 123'.isprintable()
   # True

   'dobi.123'.isprintable()
   # True

   ''.isprintable()
   # True
   ```
   - str.isspace()   
   判断字符串中是否至少有一个字符，并且所有字符都是空白字符
   ```python
   '\r\n\t'.isspace()
   True

   ''.isspace()
   False

   ' '.isspace()
   True
   ```
   - str.istitle()   
   判断字符串中的字符是否是首字母大写，其会忽视非字母字符
   ```python
   'How Python WORKS'.istitle()
   # False

   'how python works'.istitle()
   # False

   'How Python  Works'.istitle()
   # True

   ' '.istitle()
   # False

   ''.istitle()
   # False
   ```
   - str.isupper()
   ```python
   'DOBI'.isupper()
   True

   'Dobi'.isupper()
   # False
   ```

#### 6、字符串编码
   - str.encode(encoding="utf-8", errors="strict")    
   返回原字符串编码为字节串对象的版本。 默认编码为`utf-8`
   ```python
   fname = '徐'

   fname.encode('ascii')
   # UnicodeEncodeError: 'ascii' codec can't encode character '\u5f90'...

   fname.encode('ascii', 'replace')
   # b'?'

   fname.encode('ascii', 'ignore')
   # b''

   fname.encode('ascii', 'xmlcharrefreplace')
   # b'&#24464;'

   fname.encode('ascii', 'backslashreplace')
   # b'\\u5f90'
   ```

### 字符串的总结
- 1、一些引号分割的字符
- 2、不可分割的字符类型
- 3、字符串格式化操作符`%`，提供类似`printf`的功能
- 4、三引号
- 5、原始字符串对每个特殊字符串都使用它的原意
- 6、python字符串不是以`NUL`或者`'\0'`来结束的
### 引用
- [Python-3.7.4字符串方法](https://docs.python.org/zh-cn/3/library/stdtypes.html#string-methods)
- [Python 的内置字符串方法 收藏专用](https://segmentfault.com/a/1190000004598007?_ea=663877#articleHeader7)
