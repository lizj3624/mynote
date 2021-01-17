# 常用的shell命令汇总

### shell中命令间逻辑运算&& ||

#### `&&`运算符

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

#### `||`运算符

```shell
command1 || command2
```

`||`则与`&&`相反。如果`||`左边的命令（命令1）未执行成功，那么就执行`||`右边的命令（命令2）；或者换句话说，“如果这个命令执行失败了||那么就执行这个命令。

* 命令之间使用 || 连接，实现逻辑或的功能。

* 只有在`||`左边的命令返回假（命令返回值 `$? == 1`），`||` 右边的命令才会被执行。这和`c` 语言中的逻辑或语法功能相同，即实现短路逻辑或操作。

* 只要有一个命令返回真（命令返回值 `$? == 0`），后面的命令就不会被执行。

### linux下PS1、PS2、PS3、PS4

通过设置环境变量`PS1`、`PS2`、`PS3`以及`PS4`来自定义用户命令行的字符显示。如果要长期永久性修改提示符，可以将修改提示符的命令添加到`$HOME/.profile`或`$HOME/.bash_profile`文件中。

#### PS1
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

#### PS2

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

#### PS3

Shell脚本中使用select时的提示符.

你可以像下面示范的那样，用环境变量PS3定制shell脚本的select提示：
不使用PS3的脚本输出:

#### PS4

PS4-`set -x`用来修改跟踪输出的前缀

### 比较大小

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

### cut

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

## shell中执行mysql操作

### 增删改查操作
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
