- [shell中命令间逻辑运算](#shell中命令间逻辑运算)
  - [`&&`运算符](#运算符)
  - [`||`运算符](#运算符-1)
- [linux下PS1、PS2、PS3、PS4](#linux下ps1ps2ps3ps4)
  - [PS1](#ps1)
  - [PS2](#ps2)
  - [PS3](#ps3)
  - [PS4](#ps4)
- [条件判断](#条件判断)
- [cut](#cut)
- [date](#date)
- [tar](#tar)
- [shell中执行mysql操作](#shell中执行mysql操作)
- [shell中map](#shell中map)
- [shell的数组](#shell的数组)
- [shell的split](#shell的split)
- [shell简易教程](#shell简易教程)
- [sed](#sed)
- [awk](#awk)

# shell重定向

在shell脚本中，默认情况下，总是有三个文件处于打开状态，标准输入(键盘输入)、标准输出（输出到屏幕）、标准错误（也是输出到屏幕），它们分别对应的文件描述符是0，1，2 。
默认为标准输出重定向，与`1` 相同, `2>&1`意思是把标准错误输出重定向到标准输出, `&>file`意思是把标准输出和标准错误输出都重定向到文件file中
`/dev/null`是一个文件，这个文件比较特殊，所有传给它的东西它都丢弃掉

```shell
ls a.txt b.txt 1>file.out 2>file.err

# 2>&1 错误返回值传递给1输出通道, 同样&1表示1输出通道.
ls a.txt b.txt 1>file.out 2>&1
```

# shell中命令间逻辑运算

## `&&`运算符

```shell
command1  && command2
```

`&&`左边的命令（命令1）返回真(即返回`0`，成功被执行）后，`&&`右边的命令（命令2）才能够被执行；换句话说，“如果这个命令执行成功&&那么执行这个命令”。 

语法格式如下：

```shell
command1 && command2 [&& command3 ...] 
```

* 命令之间使用 && 连接，实现逻辑与的功能。

* 只有在 && 左边的命令返回真（命令返回值 $? == 0），&& 右边的命令才会被执行。

* 只要有一个命令返回假（命令返回值 $? == 1），后面的命令就不会被执行。

## `||`运算符

```shell
command1 || command2
```

`||`则与`&&`相反。如果`||`左边的命令（命令1）未执行成功，那么就执行`||`右边的命令（命令2）；或者换句话说，“如果这个命令执行失败了||那么就执行这个命令。

* 命令之间使用 || 连接，实现逻辑或的功能。

* 只有在`||`左边的命令返回假（命令返回值 `$? == 1`），`||` 右边的命令才会被执行。这和`c` 语言中的逻辑或语法功能相同，即实现短路逻辑或操作。

* 只要有一个命令返回真（命令返回值 `$? == 0`），后面的命令就不会被执行。

# linux下PS1、PS2、PS3、PS4

通过设置环境变量`PS1`、`PS2`、`PS3`以及`PS4`来自定义用户命令行的字符显示。如果要长期永久性修改提示符，可以将修改提示符的命令添加到`$HOME/.profile`或`$HOME/.bash_profile`文件中。

## PS1
`PS1`是主提示符变量,也是默认提示符变量。默认值`[\u@\h \W]\$`，显示用户主机名称工作目录。

基本上通过设置`PS1`来定义命令行提示字符即可，最常用的需求就是显示登录的用户名、主目录、主机名等等。

默认的是：

```bash
[root@centos7 ~]# echo $PS1
[\u@\h \W]\$
```

1. 如何加颜色：[加颜色链接](http://blog.csdn.net/qq_37187976/article/details/79265667)
2. 在PS1值之后加一个空格。从个人角度来讲，使用这个空格可以增加一定的可读性
3. 把定义好的变量写成脚本建议放到`/etc/profile.d/`下

PS1变量可以使用的参数值有如下：

| 参数 | 表述                                                         |
| :--- | ------------------------------------------------------------ |
| `/d` | 代表日期，格式为weekday month date，例如：”Mon Aug 1”        |
| `/H` | 完整的主机名称。例如：我的机器名称为：fc4.linux，则这个名称就是**fc4.linux** |
| `/h` | 仅取主机的第一个名字，如上例，则为fc4，.linux则被省略        |
| `/t` | 显示时间为24小时格式，如：HH：MM：SS                         |
| `/T` | 显示时间为12小时格式                                         |
| `/A` | 显示时间为24小时格式：HH：MM                                 |
| `/u` | 当前用户的账号名称                                           |
| `/v` | BASH的版本信息                                               |
| `/w` | 完整的工作目录名称。家目录会以 ~代替                         |
| `/W` | 利用basename取得工作目录名称，所以只会列出最后一个目录       |
| `/#` | 下达的第几个命令                                             |
| `/$` | 提示字符，如果是root时，提示符为：`#` ，普通用户则为：`$`    |
| `/[` | 字符”[“                                                      |
| `/]` | 字符”]”                                                      |
| `/!` | 命令行动态统计历史命令次数                                   |

## PS2

一个非常长的命令可以通过在末尾加 `\` 使其分行显示
PS2多行命令的默认提示符，默认值是 `>`

PS2一般使用于命令行里较长命令的换行提示信息，比如：

```shell
[root@centos7 ~]#echo \           
>   #默认的

[root@centos7 ~]# export PS2=">+ "  # 修改

[root@centos7 ~]#echo \            
>+   #修改后
```

当用 `\` 使长命令分行显示，非常易读。当然我也有的人不喜欢分行显示命令

## PS3

Shell脚本中使用select时的提示符.

你可以像下面示范的那样，用环境变量PS3定制shell脚本的select提示：
不使用PS3的脚本输出:

## PS4

PS4-`set -x`用来修改跟踪输出的前缀

# 条件判断

```shell
#整数比较：
-eq  等于            if [ "$a" -eq "$b" ] 
-ne  不等于          if [ "$a" -ne "$b" ]
-gt  大于            if [ "$a" -gt "$b" ]
-ge  大于等于        if [ "$a" -ge "$b" ]
-lt  小于            if [ "$a" -lt "$b" ]
-le  小于等于        if [ "$a" -le "$b" ]

<    小于（需要双括号）  (( "$a" < "$b" ))
<=   小于等于(...)      (( "$a" <= "$b" ))
>    大于(...)         (( "$a" > "$b" ))
>=   大于等于(...)      (( "$a" >= "$b" ))

#字符串比较：
=    等于           if [ "$a" = "$b" ]
==   与=等价
!=   不等于         if [ "$a" = "$b" ]
<    小于，在ASCII字母中的顺序：
     if [[ "$a" < "$b" ]]
     if [ "$a" \< "$b" ]         #需要对<进行转义
>    大于

-z   字符串为null，即长度为0
-n   字符串不为null，即长度不为0
```

# cut

cut是一个选取命令，就是将一段数据经过分析，取出我们想要的.

```shell
cut [-bn] [file] 或 cut [-c] [file] 或 cut [-df] [file]

-b ：以字节为单位进行分割。这些字节位置将忽略多字节字符边界，除非也指定了 -n 标志。
-c ：以字符为单位进行分割。
-d ：自定义分隔符，默认为制表符。
-f ：与-d一起使用，指定显示哪个区域。
-n ：取消分割多字节字符。

## 常用的命令
$ who
rocrocket :0 2009-01-08 11:07
rocrocket pts/0 2009-01-08 11:23 (:0.0)
rocrocket pts/1 2009-01-08 14:15 (:0.0)

$ who|cut -b 3
c
c
c

who|cut -b 3-5,8
croe
croe
croe

##但有一点要注意，cut命令如果使用了-b选项，那么执行此命令时，cut会先把-b后面所有的定位进行从小到大排序，然后再提取。可不能颠倒定位的顺序哦。这个例子就可以说明这个问题：
$ who|cut -b 8,3-5
croe
croe
croe

$ who|cut -b -3
roc
roc
roc

$ cat cut_ch.txt
星期一
星期二
星期三
星期四

$ cut -c 3 cut_ch.txt
一
二
三
四
```

# date
```shell
## 1、获取今天日期

$ date -d now +%Y-%m-%d   #或者
$ date +%F

#2、获取明天日期
$ date -d next-day +%Y-%m-%d
$ date -d tomorrow +%Y-%m-%d

# 3、获取昨天日期
$ date -d yesterday +%Y-%m-%d  #或者
$ date -d last-day +%Y-%m-%d  #或者
$ date -d "1 days ago" +%Y-%m-%d 
##"n days ago"  表示n天前的那一天

# 4、获取取30天前的日期
$ date -d "30 days ago" +%Y-%m-%d  

# 5、使用负数以得到相反的日期
$ date -d 'dec 14 -2 weeks' +%F   #相对于dec 14这个日期的两周前的日期
$ date -d '-100 days' +%F         #100天以前的日期
$ date -d '50 days' +%F           #50天后的日期

# 6、时间戳
$ date '+%s'
1327312578

# 7、时间转换
$ date -d "1970-01-01 956684800 sec GMT"
Tue Apr 25 10:46:40 PDT 2000

$ date -d "2000-01-01 GMT" '+%s'
946684800

# 扩展：
$ date -d next-month +%F   #下个月今天日期
$ date -d last-month +%F   #上个月今天日期
$ date -d next-year +%Y    #明年日期
$ date -d '2 weeks' +%F    #获取两星期以后的日期
```

# tar
```shell
#解包到指定的目录
tar zxvf filename.tar.gz -C /specific dir

# 压缩到指定目录
tar zcvf /specific/filename.tar.gz filename
``` 
# shell中执行mysql操作
```shell
#!/usr/bin/env bash
HOST="localhost"
PORT="3306"
USER="xiaohai"
PASSWD="xiaohai"
DATABASE="testdb"

# 执行sql 无需获取返回值，sql执行失败则脚本异常结束
# 参数1 完整的sql语句
function mysqlExecute {
  mysql -u"${HOST}" -P"${PORT}" -u"${USER}" -p"${PASSWD}" -D"${DATABASE}" -e "$1"
  if [[ $? -eq 0 ]]
  then
      echo "exec sql succeed: "
      echo "$1"
  else
      echo "exec sql failed: "
      echo "$1"
      exit -1
  fi
}

# 执行sql 需获取返回值，sql执行失败则脚本异常结束
# 参数1 完整的select语句
function mysqlExecuteQuery {
  # 返回结果：-e带表头 -Ne不带表头
  rs=(`mysql -u"${HOST}" -P"${PORT}" -u"${USER}" -p"${PASSWD}" -D"${DATABASE}" -Ne "$1"`)
  if [[ $? -eq 0 ]]
  then
      # 打印查询结果中的每一个元素
      echo ${rs[*]}
  else
      echo "exec sql failed: "
      echo "$1"
      exit -1
  fi
}

# 获取指定行指定列的值
# 参数1 字段所在行数
# 参数2 字段所在列数
# 参数3 select总列数
# 参数4+ 查询结果数组
function getValueFromResult {
  local rowIndex
  local colIndex
  local column_num
  local rs
  rowIndex=$1
  colIndex=$2
  column_num=$3
  rs=(`echo "$@"`)
  # 下标=总列数*(第几行-1)+第几列-1+非查询结果的其他参数个数
  idx=$[$column_num*($rowIndex-1)+$colIndex-1+3]
  if [[ $[idx] -le ${#rs[@]} ]]
  then
      # 根据下标获取目标结果
      echo ${rs[$idx]}
  fi
}

# 计算查询返回结果数据行数
# 参数1 select总列数
# 参数2 查询结果数组
function getRowNumFromResult {
    local rs
    rs=(`echo "$@"`)
    echo $[(${#rs[@]}-1)/$1]
}

# select列数
column_num=2

selectSql="select id, name from test;"

# 调用方法执行sql，打印出sql执行结果但不获取返回值
mysqlExecute "$selectSql"
# 用数组接收查询返回值
result=(`mysqlExecuteQuery "$selectSql"`)

# 计算查询返回结果数据行数
row_num=`getRowNumFromResult ${column_num} ${result[*]}`

for (( i=1; i<=$row_num; i=i+1))
do
    # 获取第一列的值
    id=`getValueFromResult $[i] 1 $column_num ${result[*]}`
    # 获取第二列的值
    name=`getValueFromResult $[i] 2 $column_num ${result[*]}`
    echo "id: $id, name: $name"
done
```

# shell中map
在使用map时，需要先声明，否则结果可能与预期不同，array可以不声明
```shell
declare -A myMap
myMap["my03"]="03"

declare -A myMap=(["my01"]="01" ["my02"]="02")

# 1）输出所有的key
#若未使用declare声明map，则此处将输出0，与预期输出不符，此处输出语句格式比arry多了一个！
echo ${!myMap[@]}

#2）输出所有value
#与array输出格式相同
echo ${myMap[@]}

#3）输出map长度
#与array输出格式相同
echo ${#myMap[@]}

#1)遍历，根据key找到对应的value
for key in ${!myMap[*]};do
    echo $key
    echo ${myMap[$key]}
done

#2)遍历所有的key
for key in ${!myMap[@]};do
    echo $key
    echo ${myMap[$key]}
done

#3)遍历所有的value
for val in ${myMap[@]};do
    echo $val
done
```

# shell的数组
Bash Shell 只支持一维数组（不支持多维数组），初始化时不需要定义数组大小，数组元素的下标由0开始，Shell 数组用括号来表示，元素用"空格"符号分割开，语法格式如下：
```shell
# 定义 array_name=(value1 value2 ... valuen)
my_array=(A B "C" D)
array_name[0]=value0
array_name[1]=value1
array_name[2]=value2

# 读取数组
my_array=(A B "C" D)

echo "第一个元素为: ${my_array[0]}"
echo "第二个元素为: ${my_array[1]}"
echo "第三个元素为: ${my_array[2]}"
echo "第四个元素为: ${my_array[3]}"

# 获取数组中的所有元素
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组的元素为: ${my_array[*]}"
echo "数组的元素为: ${my_array[@]}"

# 数组的长度
my_array[0]=A
my_array[1]=B
my_array[2]=C
my_array[3]=D

echo "数组元素个数为: ${#my_array[*]}"
echo "数组元素个数为: ${#my_array[@]}"

# 变量数组
for(( i=0;i<${#array[@]};i++)) 
#${#array[@]}获取数组长度用于循环
do
    echo ${array[i]};
done;

for element in ${array[@]}
#也可以写成for element in ${array[*]}
do
    echo $element
done
```
# shell的split
```shell
# 方法1
IN="bla@some.com;john@home.com"

mails=$(echo $IN | tr ";" "\n")

for addr in $mails
do
    echo "> [$addr]"
done

# 方法2
IN="bla@some.com;john@home.com"

OIFS=$IFS
IFS=';'
mails2=$IN
for x in $mails2
do
    echo "> [$x]"
done

IFS=$OIFS

# 方法3
IFS=';' read -ra ADDR <<< "$IN"
for i in "${ADDR[@]}"; do
    echo "$i"
done
```

# shell简易教程
[shell简易教程](https://github.com/lizj3624/mynote/blob/master/dev-lang/shell/shell%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B.md)
[运维必知必会：Bash Shell 脚本的实践指南](https://mp.weixin.qq.com/s/SsnILIBqDyo24YIflbFNDw)
# sed
[sed教程以及常用命令](https://github.com/lizj3624/mynote/blob/master/dev-lang/shell/sed-cmd.md)
# awk
[awk简易教程](https://github.com/lizj3624/mynote/blob/master/dev-lang/shell/awk%E7%AE%80%E6%98%93%E6%95%99%E7%A8%8B.md)
