# 1. 语法
### 变量
1. `${variable}`获取变量值，简写`$variable`。当涉及变量拼接时，必须使用{}。如：`${variable}_name`。
1. `variable=value`变量赋值，=左右两边不能有空格。
1. 命令结果赋值。`variable=$(ls -a)`或者`varivale=${ls -a}`。
1. 环境变量。打开shell的时候，创建环境变量。该shell创建的子进程将继承该shell的环境变量。export命令可以设置环境变量，供子进程继承使用。但子进程不能export给父进程使用。
1. 位置变量。`$0`代表脚本名字，`$1`代表第1个参数，`$n`代表第n个参数。
1. 特殊符号变量。`$@`与`$*`表示所有参数；`$#`表示参数个数。
1. `$@` 与 `$*` 区别： 加入双引号后，*表示的参数不会被IFS分隔。
2. 特殊变量

名称 | 说明
---|---
$0 | 脚本名称
$1-9 | 脚本执行时的参数1到参数9
$? |	脚本的返回值　　　　
$# |	脚本执行时，输入的参数的个数
$@ |	输入的参数的具体内容（将输入的参数作为一个多个对象，即是所有参数的一个列表）
$* | 输入的参数的具体内容（将输入的参数作为一个单词）


```shell
bash test.sh 1 2
------------------
code> for item in "$*"; do echo $item; done
out > 1 2

code> for item in "$@"; do echo $item; done
out > 1
      2
```

### 操作符
1. = 赋值操作符，操作符两端不能有空格。
1. (())类似于let命令，允许算是操作，同时允许类似C语言的表达式
```shell
(( a++ ))
(( t = a < 45 ? 4 : 5 ))
(( i = 1;  i < 10; i++ ))
```
### 数组
1. 数组赋值`array[key]=value`或`array = (value1 value2 value3)`。
1. 访问数组`${array[key]}`
1. 删除元素`unset array[key]`
1. 删除数组`unset array`
1. 数组长度`${#array[*]}`或`${#array[@]}`
1. 数组展开`${array[*]}`或`${array[@]}`
```shell
array=(1 2 3)
for item in ${array[*]}
do
    echo $item
done
```
### 退出状态码
1. 取值范围0-255之间整数，成功返回0，失败返回非0。
1. exit 等价于 exit $?。而 $? 取上一条命令的返回状态码
1. ! 不影响命令的执行，只影响命令的状态码。
```shell
code> ! echo "hello world"; echo $?
out > 1
```
NOTE: ! 改变返回状态码的值。
### 测试条件
1. if 测试退出状态码，0代表成功，1代表失败。
1. test 与 [] 与[[]]
1. -a 逻辑与
1. -o 逻辑或
```shell
if [ "$expr1" -a "$expr2" ]  ==> expr1 与 expr2 同时为真
if [[ "$expr1" && "$expr2" ]] ==> [[]] 可以采用&&, ||
```
```shell
    #!/bin/sh
    a=10
    b=20
    if [ $a == $b ]
    then
       echo "a is equal to b"
    elif [ $a -gt $b ]
    then
       echo "a is greater than b"
    elif [ $a -lt $b ]
    then
       echo "a is less than b"
    else
       echo "None of the condition met"
    fi
```
文件测试
符号 | 含义
---|---
-d	file|为目录且存在
-e	file|为文件且存在
-f	file|为非目录普通文件且存在
-s	file|存在且长度不为 0
-L	file|为连接且存在
-r	file|为文件且可读
-w	file|为文件且可写
-x	file|为文件且可执行

数字测试
符号 | 含义
---|---
-eq|equal
-ne|not equql
-gt|greater than
-ge|greater equal
-lt|less than
-le|less equal

双圆括号 `(($a < $b))`；符号：`<, <=, >, >=,==，!=`

字符串测试
`=` 仅仅是对等号两边的字符串进行逐字匹配，等号两端必须有空格，否则相当于字符串连接了。 
`==` 在[]中的表现同`=`，但如果采用`[[]]`，则展现不同。
```shell
[[ "$a" == a* ]] 相当于正则，匹配以a开头的字符串
[[ "$a" == "a*" ]] 关闭正则，逐字匹配

code> a="aabb"; if [[ $a == a* ]]; then echo "got"; else echo "not"; fi
out > got

code> a="aabb"; if [[ $a == "a*" ]]; then echo "got"; else echo "not"; fi
out > not
```
`!=` 同`==`使用方式，判断不等。 
`<` 与 `>` 通过ascll码进行大小比较。
`[]` 中必须转义，因为`[]`相当于shell表达式，不转义，相当于重定向。
```shell
[[ "$a" < "$b" ]] 或 [ "$a" \< "$b" ]
```
`-z` 字符串为空 
`-n` 字符串非空。 
`if [ -z "$a" ]`
`[]` 与 `[[]]` 
`[]` 相当于`test`命令，为shell的一个内置命令。 
`[[]]` 为一个关键字，非命令。
### 循环与分支
1. `for arg in [list]; do command; done`
1. `for ((i=min; i <= max; i+=step)) do command; done`
1. `while [condition] do command; done`
1. `until [ condition ] do command done`
1. `casc "$variable" in "$condition") command;; esac`
```shell
var="spch2008"
case $var in
    "spch2009" | "spch2010") echo "condition 1";;
     spch*) echo "condition2";;
     *) echo "last condition";;
esac
```
### 引号
`shell`中输入命令，得到一个命令行，该命令行被`shell`解析。对于一个命令行中的字符，`shell`将其分成两种。一种是普通文字，一种是元字符，对shell来说，具有特定功能的保留字。

`单引号`:单引号内所有元字符被关闭。
`双引号`:双引号大部分元字符功能被关闭，仅保留$,`,\三种。
`反斜线`:\之后的单一元字符功能被关闭。

`IFS`域分隔符，由三者组成。`IFS`用于拆分command line中的每个单词。如果要还原字符的本义，即关闭元字符功能，则需要使用引号。
```shell
code> line="one two three"; for word in $line; do echo $word; done
out > one
      two
      three

code> line="one two three"; for word in "$line"; do echo $word; done
out > one two three
```
### 命令替换
` 与 () ：可将一个命令的输出转入另一个上下文中；也可以作为另一个命令的参数；也可以用来设置变量；还可以为for循环产生参数列表。
```shell
files=`ls *.txt`
files=$(ls *.txt)

variable=`cat file`
variable=`<file`

for file in `ls *.sh`; do echo $file; done
```

### 操作变量
1. 取长度 `${#string}, ${#array}`
2. 截取 `${string:position:len}`
```shell
string=spch2008
echo ${string:4:4}
echo ${string:4}         #位置4到最后
echo ${string:(-4)}     #从后向前计算索引，负数加括号或则空格
echo ${string: -4}

out > 2008
```
### 函数及返回值
1. 函数定义：`function name() { }` 
1. 返回数字：`return`
```shell
function name()
{
    return 100
}

name
echo $?   #取得返回值
```
### 特殊变量
`.` 类似于C中的`#include`，引入shell文件
```shell
. func.sh # 有空格，引入另一个shell文件
```
- `:` 空语句 
- `$$` 当前脚本的进程ID 
- `$!` 最近一个执行命令的后台进程ID
```shell
echo $$
./a.out &
echo $!

out > 18180 和 18181
```
`()` 命令集，展开一个新的进程执行命令集。

### 注释
单行采用`#`，若一行不足以写完，另一行以`#+`代表承接上一行
```shell
#  one line
#+ one line more
```
### 算术操作
整数操作 
`let` shell内置操作，计算表达式值 
`expr`一个命令，类似let，用于计算表达式值，但`expr`直接输出结果，而不是保存在变量中。
```shell
num_a=10
let result=num_a*2
echo $result
expr $num_a \* 2
```
NOTE: `expr` 操作符两侧必须留有空格；`*`必须被转义，因为`*`被shell展开，因此需要转义，将`*`传递给`expr`。 
- 浮点操作
```shell
echo "3*2" | bc
echo "$num * 3" | bc

code> echo "scale=2; 3/4" | bc     # 设置精度
out > .75
```
# 2. 命令
### find
文件查找
1. 按名查找 `-name`， 忽略大小写 `-iname`
   ```shell
   find path -iname Spch
   ```
1. 多表达式 。 满足一个即可。and: `-a` ; or `-o`
   ```shell
   find path \( -name "*.txt" -o -name "*.t" \)
   ```
2. 指定路径范围 `-path`
   ```shell
   find . -path "*/linux/*"    #路径中包含linux
   ```
1. 正则表达式 `-regex`
   ```shell
   find . -regex ".*\(\.py|\.sh\)$"   #查找py或sh结尾的文件
   ```
   默认采用emacs。 正则表达式种类繁多，建议通过-regextype 指定采用的正则类型。 
posix-basic 和 posix-extend。参见正则
2. 取反 `!`
   ```shell
   find . ! -name "*.txt"
   ```
1. 文件类型 -type   
   f 常规文件 l 符号链接 d 文件夹 c 字符设备 b 块设备 s 套接字
1. 访问时间
- `atime access time` 访问时间
- `mtime modification time` 修改时间
- `ctime metadata` 修改时间 (权限，拥有者等)
- `amin mmin cmin` 以分钟为单位
```shell
find . -atime -7   #7天之内被访问过的
find . -atime 7     #距离访问时间刚好7天
find . -atime +7   #7天之前被访问过的
```
8. 基于文件大小 `-size`
- `-size 2k` 等于2k
- `-size +2k` 大于2k
- `-size -2k` 小于2k
- b 比特 c 字节 w 2字节 k kb m mb g gb
9. 基于文件权限 `-perm`
- perm -> permission， 权限为可读可写可执行 777
```shell
find . -type f -perm 644
```
10. 基于用户 `-user`
11. 查找删除 `-delete`
```shell
find  . -type f -name "spch.txt" -delete
```
12. 嵌套层次 `-maxdepth -mindepth`
```shell
find . -maxdepth 2 -name "*.txt"  #只递归两层文件夹进行查找
```
NOTE: 参数优先级，左边大于右边， 如果先指定名字，在指定嵌套深度， 
则，依然会全部递归，找到名字后，在比较嵌套层次
13. 查找 + 动作 `exec`
    ```shell
    find . -user spch2008 -type -f -exec chown linux {} \;  # 将属于spch2008的文件修改所有者为linux
    ```
    {} 表示查找到的文件； 表达式以分号结尾，分号需要转义
    
    `find …… -exec cat {} \; > a.txt`
    
    find: 整个find命令为一个输出流，因为不需要使用>>追加操作。
    
    ```shell
    find …… -exec ./command.sh {} \;
    ```
### tr
tr [ option ] set1 set2 =》 将set1集合内容替换为set2集合内容，set1与set2一一对应
1. 指定范围 (将a-c替换为x-z)
```shell
cat text | tr 'a-c' 'x-z'
```
2. 删除 `-d`
```shell
tr -d '0-9'  #删除0-9
```
3. 去重连续重复字符 `-s`
   ```shell
   cat  text | tr -s 'a'
   cat  text | tr -s 'a-z'   #删除text中a-z重复的字符
   cat  text | tr -s '\n'     #删除联系空行
   ```
   
### sort
1. 输出排序结果到源文件 `-o`
   ```shell
   cat file | sort -o file
   ```
2. 以数字而非ascll进行排序比较 `-n`
3. 从大至小进行排序 `-r`
4. 合并两个已排序文件 `-m`
   ```shell
   sort -m file1 file1 #合并到file1中
   ```
5. 检查文件是否已经按从小到大排序`-C`, `$?` 为0，表明已排序。
6. 多列，指定排序列（默认以`\t`作为分隔）。`-t` 指定分隔符。 (cunyi)
```shell
mac 1000
winxp 500

sort -k 2 -n data.txt 
```
7. 指定排序内容 `-k pos1, pos2`
```shell
 2134
 3404
 --- sort -k2,3 ----
```
8. 忽略每行前空格 `-b`
### uniq
1. 去重。处理数据之前，必须先排序。
1. 去除重复行。 `uniq data.txt`
1. 只显示不重复行 `-u`
1. 只显示重复行 `-d`
1. 去除重复行，且显示行数 `-c`
```shell
  tom
  tom
  rom
 --- uniq -c --------------
  2 tom
  1 rom
```
5. 指定比较字段。 `-s` 起始位置 `-w`匹配长度
### cut
提取文件内容
1. 提取指定字段(默认tab分隔) `-f`
   ```shell
   tom 18 studnet
   ---- cut -f 2,3 file ---
   18 student
   ```
2. 排除某个字段，剩下全要 `--complement`
   ```shell
   cut -f3 --complement file
   ```
3. 指定分隔符 `-d`
4. 指定字符或字节范围
-    `-b` 字节
-    `-c` 字符
-    `-f` 域
- abcdef ==> cut -c1-5 ==> 截取abcde
### wc
统计字符
- -l 统计行数
- -w 统计单词
- -c 统计字符
- -L 最长行的字符数
- wc file 总览，输出行数，单词数，字符数
### tail
打印文件结尾部分
- tile file 打印尾10行
- tile -n file 打印尾n行
- tile -f file 打印最新的，即文件在动态增长
- tile -f file –pid pid 指定进程死亡，tail命令结束

### tar
打包工具
- 压包
`tac -cf output.tar file1 file2`
c (create) f (filename)

- 看包
`tar -tf output.tar`
`tar -tvf output.tar` 文件详细信息，如大小等。

- 追加
`tar -rf output.tar newfile`

- 解包
`tar -xf output.tar`

- 抽取
`tar -xf output.tar file1 file2`

- 合并
`tar -Af tar1 tar2` #合并到tar1中

- 更新
`tar -uf output.tar file1`

- 删除
`tar -f output.tar –delete file1`

- 压缩
`-j bzip2 .bz2`
`-z gzip .gz`

① 根据后缀自动选取压缩工具 -a

tar -acf output.tar.gz file1 file2

② 指定压缩工具

tar -zcf ouput.tar.gz file1 file2

解压缩

① 自动选择解压工具

`tar -xaf output.tar.gz -C `路径

② 指定压缩工具
`tar -xzf output.tar.gz`

### du
查看磁盘利用率
```shell
df -h
```

### time
查看某个命令的执行时间
- time ls 查看ls执行时间
- -o 将信息写入文件
- -o -a 以追加方式写入文件

### crontab
预设时间，执行脚本。
- 六个区间 

Minute (0-59) Hour (0-23) Day (1-31) Month (1-12) Weekday (0-6) Command

* 代表这个区间段执行 

`如 00 02 * * * ，每天两点执行； /2 * * * 每两分钟执行一次； 00 5,6,7 * * * 每天5，6，7点执行。`
```shell
编写文件：vim task.cron
编写命令：00 02 * * * /home/script.sh
执行：chrontab task.cron
```

- 查看crontab
```shell
crontab -l
crontab -l -u spch2008  #查看指定用户的crontab
```

- 删除
```shell
crontab -r                  #删除全部
crontab -r -u spch2008      #删除某个用户的全部
```

# 3. 知识点
重定向
文件描述符： 输入`0`， 输出`1`, 错误输出`2`。

`2>&1`不可以写成`2>1`， 错误写法，相当于把错误输出打印到文件1中，而不是重定向。加入&告诉shell，后面的是一个文件描述法，而非文件名。

块重定向
`>，>>，<，<<`可以为单个命令进行重定向，也可以通过`{}`与`()`进行块的重定向。

# 4. grep and sed 
[grep&sed](https://blog.csdn.net/spch2008/article/details/50801729)

# 5. shell 
[Shell 程序设计教程](http://kuanghy.github.io/shell-tutorial/index.html)