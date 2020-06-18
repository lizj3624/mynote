### Python中json使用详解
#### json概念
- json是一种通用的数据类型，任何语言都认识
- 接口返回的数据类型都是json
- 长得像字典，形式也是k-v { }
- 其实json是字符串
- 字符串不能用key、value来取值，要先转成字典才可以
- [json官方](http://www.json.org/)
- 格式如下：
```json
{
    "error_code": 0, 
    "stu_info": [
        {
            "id": 309,
            "name": "小白",
            "sex": "男",
            "age": 28,
            "addr": "河南省济源市北海大道32号",
            "grade": "天蝎座",
            "phone": "18512572946",
            "gold": 100
        },
        {
            "id": 310,
            "name": "小白",
            "sex": "男",
            "age": 28,
            "addr": "河南省济源市北海大道32号",
            "grade": "天蝎座",
            "phone": "0000000",
            "gold": 100
        }
    ]
}
```
#### json串转成字典
##### json.loads()方法
```python
import json  #引用json模块
res = json.loads(s)
print(res)  #打印字典
print(type(res))  #打印res类型
print(res.keys())  #打印字典的所有Key
```
- 要先读文件，然后再转换
```python
f = open('stus.json', encoding='utf-8')
content = f.read()   #使用loads()方法，需要先读文件
user_dic = json.loads(content)
print(user_dic)

### with
with open('stat.json') as f:
    content = f.readlines()
    user_dict = json.loads(content)
```
##### json.load()方法
```python
import json
f = open('stus.json', encoding='utf-8')
user_dic = json.load(f)
print(user_dic)
```
##### 区别
- `loads()`传的是字符串，而`load()`传的是文件对象
- 使用`loads()`时需要先读文件再使用，而`load()`则不用

#### 字典转成json串
- 文件里只能写字符串，但可以把字典转成json串，json串是字符串，可以存到文件里
##### json.dumps()方法
```python
stus={'xiaojun':'123456','xiaohei':'7891','abc':'11111'}
#先把字典转成json
res2=json.dumps(stus)
print(res2)#打印字符串
print(type(res2))#打印res2类型
```
- json.dumps()方法：把字典转成json串
```python
with open('stus.txt','w',encoding='utf-8' as f:   #打开文件
    f.write(res2)#在文件里写入转成的json串
```
- 使用.dumps()方法前，要先打开文件，再写入
```python
stus = {'xiaojun':'123456','xiaohei':'7890','lrx':'111111'}
res2 = json.dumps(stus,indent=8,ensure_ascii=False)
print(res2)
with open('stus.json','w',encoding='utf-8') as f:  #使用.dumps()方法时，要写入
    f.write(res2)
```
##### json.dump()方法
```python
stus={'xiaojun':'123456','xiaohei':'7890','lrx':'111111'}
f=open('stus2.json','w',encoding='utf-8')
json.dump(stus,f,indent=4,ensure_ascii=False)
```
##### 区别
- `json.dump()`不需要使用`.write()`方法，只需要写哪个字典、哪个文件即可；而`.dumps()`需要使用`.write()`方法写入
- 如果要把字典写到文件里面的时候，`dump()`好用；但如果不需要操作文件，或需要把内容存到数据库和`Excel`，则需要使用`dumps()`先把字典转成字符串，再写入

##### dump和dumps参数
- `json.dumps\dump`中使用参数`indent`，为字符串换行`+`缩进：
```python
res2=json.dumps(stus.indent=4)
print(res2) #打印字符串
#结果为： 
{
    "xiaojun": "123456",
    "xiaohei": "7891",
    "lrx": "hailong",
    "tanailing": "111111"
}
```
- `.dumps\dump`中使用参数`ensure_ascii`，为内容输出为中文
```python
res2 = json.dumps(stus, indent=4, ensure_ascii=False)  #为False时内容输出显示正常的中文，而不是转码
print(res2)
```
- **不管是dump还是load，带s的都是和字符串相关的，不带s的都是和文件相关的**

#### 引用
- [原文](https://www.cnblogs.com/yanwuliu/p/9593826.html)
- [python官方json文档](https://docs.python.org/zh-cn/3/library/json.html)
