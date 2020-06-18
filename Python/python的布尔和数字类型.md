### 布尔类型
- 布尔只有两个值，True、False
- 如下情况被视为False    
  1）被定义为假值的常量: `None` 和 `False`。

  2）任何数值类型的零: `0, 0.0, 0j, Decimal(0), Fraction(0, 1)`

  3）空的序列和多项集: `'', (), [], {}, set(), range(0)`

- 布尔运算    
  `and、or、not`   

运算 | 结果 | 备注
---|---|---
`x or y` | if x is false, then y, else x | (1)
`x and y` | if x is false, then x, else y | (2)
not x | if x is false, then True, else False | (3)
注释: 

这是个短路运算符，因此只有在第一个参数为假值时才会对第二个参数求值。

这是个短路运算符，因此只有在第一个参数为真值时才会对第二个参数求值。

not 的优先级比非布尔运算符低，因此 `not a == b` 会被解读为 `not (a == b)` 而 `a == not b` 会引发语法错误。

### 数字类型
#### 整型(int)    
 整数具有无限的精度
#### 浮点型(float)
浮点数通常使用 C 中的 double 来实现
#### 复数(complex)
复数包含实部和虚部，分别以一个浮点数表示。 要从一个复数 z 中提取这两个部分，可使用 z.real 和 z.imag

#### 引用
- [python官方文档](https://docs.python.org/zh-cn/3/library/stdtypes.html#numeric-types-int-float-complex)