### Python中的转义
在Python交互式解释器中，输出的字符串会用引号引起来，特殊字符会用反斜杠(`\`)转义。    
如果遇到带有`\`的字符被当作特殊字符时，有以下两种处理方法：    
- 使用双反斜杠(`\\`)来转义
- 使用原始字符串，方法是在第一个引号前面加上一个`r`

```python
#1.非转义输出
>>> print 'c:\name.txt'
c:
ame.txt
>>> print 'c:\test.txt'
c:    est.txt

#2.反斜杠(\)转义输出
>>> print 'c:\\name.txt'
c:\name.txt
>>> print 'c:\\test.txt'
c:\test.txt

#3.使用原始字符串输出
>>> print r'c:\name.txt'
c:\name.txt
>>> print r'c:\test.txt'
c:\test.txt
```

转义字符 | 说明
---|---
\	| 在行尾的续行符，即一行未完，转到下一行继续写
\'	| 单引号
\"	| 双引号
\0	| 空
\n	| 换行符
\r	| 回车符
\t	| 水平制表符，用于横向跳到下一制表位
\a	| 响铃
\b	| 退格（Backspace）
`\\`	| 反斜线
\0dd	| 八进制数，dd 代表字符，如 \012 代表换行
\xhh	| 十六进制数，hh 代表字符，如 \x0a 代表换行

