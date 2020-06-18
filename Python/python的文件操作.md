### python的文件操作
#### 文件打开模式
- r ，只读模式【默认模式，文件必须存在，不存在则抛出异常】
- w，只写模式【不可读；不存在则创建；存在则清空内容】
- x， 只写模式【不可读；不存在则创建，存在则报错】
- a， 追加模式【可读； 不存在则创建；存在则只追加内容】，文件指针自动移到文件尾   

"+"表示可以同时读写某个文件    
- r +， 读写【可读，可写】
- w +，写读【可读，可写】，消除文件内容，然后以读写方式打开文件。
- x + ，写读【可读，可写】
- a +， 写读【可读，可写】，以读写方式打开文件，并把文件指针移到文件尾    

"b"表示以字节的方式操作，以二进制模式打开文件，而不是以文本模式。
- rb或r+b
- wb或w+b
- xb或w+b
- ab或a+b    
注： 以b方式打开时，读取到的内容是字节类型，写入时也需要提供字节类型，不能指定编码

#### 读取txt文件
```python
with open(text_path,'a+') as my_file:
    #values2 = my_file.readlines()
    #values2 = my_file.read()
    values2 = my_file.readline()
    
# read() 读取整个文件。返回str
# readline() 读取一行数据。返回str
# readlines() 读取所有行的数据。返回list

#遍历读取文件的每一行，如果文件有路径时，要加'r'，读取完数据后关闭close
with open(r'H:\git\py\pystr\mystr.txt', 'r') as my_file:
    for line in my_file:
        print(line)
    my_file.close()    
```

#### 读取csv文件
```python
import csv 
with open('test_data/user_info.csv','r+') as my_file:
    data = csv.reader(my_file)
    print(type(data))  #返回对象
    for row in data:
        print(row)     #返回list
```

#### 读取excel文件
读取Excel文件需要安装xlrd库，在cmd界面输入命令：`pip install xlrd`
```python
import xlrd
# 打开Excel文件读取数据
workbook = xlrd.open_workbook('a.xlsx')
# 打印所有的sheet列出所有的sheet名字
print(workbook.sheet_names())
# 根据sheet索引或者名称获取sheet内容
Data_sheet = workbook.sheets()[0]
# Data_sheet = workbook.sheet_by_index(1)
# Data_sheet  = workbook.sheet_by_name(u'Charts')
# 获取sheet名称、行数和列数
print(Data_sheet.name,Data_sheet.nrows,Data_sheet.ncols)

# 获取整行和整列的值（列表）
rows = Data_sheet.row_values(0) #获取第一行内容
cols = Data_sheet.col_values(1) #获取第二列内容
print(rows)
print(cols)

# 获取单元格内容的数据类型
# 相当于在一个二维矩阵中取值
# （row,col）-->(行,列)
cell_A1 = Data_sheet.cell(0,0).value# 第一行第一列坐标A1的单元格数据
# cell_C1 = Data_sheet.cell(0,2).value  # 第一行第三列坐标C1的单元格数据

# 检查单元格的数据类型
# ctype的取值含义
# ctype : 0 empty,1 string, 2 number, 3 date, 4 boolean, 5 error
print(Data_sheet.cell(4,0).ctype)

#栗子
def Get_Excel(file_path):
    all_case = []
    workbook = xlrd.open_workbook(file_path)
    sheet_data = workbook.sheet_by_index(1)
    #获取总行数
    row_sum = sheet_data.nrows
    #起始为1，所以不包括表头
    for i in range(1,row_sum):
        is_activity = sheet_data.cell(i,9).value
        #过滤不运行的测试数据
        if int(is_activity) == 1:
            all_case.append({
                "case_name":sheet_data.cell(i,4).value,
                "data":sheet_data.cell(i,5).value,
                "url":sheet_data.cell(i,6).value,
                "method":sheet_data.cell(i,7).value,
                "code":sheet_data.cell(i,8).value
            })
    return all_case
```

#### 读取yaml文件
Python读取yaml文件需要安装第三方库pyyaml，cmd界面输入命令：`pip install pyyaml`
config.yaml文件内容如下：
```python
#url : https://www.baidu.com
#email : ['123456@qq.com','admin@qq.com']
#DB :
#  host: localhost
#  name : test
#  user : admin
#  password : admin
  
import yaml
with open("config/config.yaml","r+",encoding='utf-8') as my_yaml:
    yaml_data = yaml.load(my_yaml)
    print(yaml_data)
    print(type(yaml_data))   #返回字典
    print(type(yaml_data['url']))
    print(type(yaml_data['email']))
    print(type(yaml_data['DB']))
    
###输出
{'url': 'https://www.baidu.com', 'email': ['123456@qq.com', 'admin@qq.com'],
 'DB': {'host': 'localhost', 'name': 'test', 'user': 'admin', 'password': 'admin'}}
<class 'dict'>
<class 'str'>
<class 'list'>
<class 'dict'>
```

#### 文件对象的内置方法和属性
```python
file.close()
file.flush()
file.read()
file.write()

#对象属性
file.name
file.encoding
file.mode
```

#### 文件系统在python的实现
文件系统的操作大部分都是通过python的`os`模块实现，该模块是python访问操作系统的主要接口，os模块已经屏蔽了`linux, windows，mac`等系统的差异，在各个系统中直接导入`os`模块就可以，不用是什么操作系统。
```python
remove()/unlink()  #删除文件
rename()/renames() #重命名文件
chdir()            #改变当前的工作目录
chroot()           #改变当前进程的根目录
mkdir()            #创建目录
```
[详细参考python官方的os文档](https://docs.python.org/3/library/os.html?highlight=os#module-os)   
`os.path`可以对路径名称操作，可以对路径操作
```python
join()    #路径合并
split()   #路径切分
exist()   #指定的路径是否存在
```
[详细参考一下`os.path`的官方文档](https://docs.python.org/zh-cn/3/library/os.path.html)