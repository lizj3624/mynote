- [1、用sed实现路径替换](#1用sed实现路径替换)
- [2、sed几种替换用法](#2sed几种替换用法)
  - [通过变量替换](#通过变量替换)
  - [替换空格/tab](#替换空格tab)
- [3. sed同时替换多个变量](#3-sed同时替换多个变量)
# 1、用sed实现路径替换
```shell
sed "s:${SRC_PATH}:${DST_PATH}:g" $filename
```
一般情况下，`sed 's/pattern/pattern/flag'`是用`/`来进行分隔的。但这里的源字符串和替换字符串中，都带有字符`/`，如果再用`/`进行分隔，就会产生冲突。所以改用了`：`进行分隔。

# 2、sed几种替换用法
## 通过变量替换
```shell
a="one"
b="two"
# 第一种：
eval sed -i ’s/$a/$b/’ filename

# 第二种（推荐）:支持shell变量
sed -i "s/$a/$b/" filename

# 第三种：
sed -i ’s/’$a’/’$b’/’ filename 

# 第四种：
sed -i s/$a/$b/ filename
```
如果变量`a`或者`b`有` -、/` 等字符要用 `\` 进行转义
字符串变量中可以用单引号或者双引号 ，区别`：`双引号支持变量引用、转义符（比如`\n` 换行），单引号不支持
`sed` 命令执行时加 `-i` 参数会把修改应用到源文件上，否则只是屏幕显示

## 替换空格/tab
```shell
# 把文件filename中的a字符换成A字符

sed -i "s/a/A/g" filename

# 批量替换 替换dir文件夹下所有文件中的a字符变成A字符

sed -i "s/a/A/g" `grep a -rl dir/`

# 替换为空格 将tab替换为空格

sed -i "s/\t/    /g" filename

# 将空格替换成,
cat word.txt | sed 's/[ ][ ]*/,/g'
# s代表替换指令；
# 每个[ ]都包含有一个空格；
# *号代表0个或多个；
# g代表替换每行的所有匹配；

cat word.txt | sed 's/\s\+/,/g' 
#其中\s代表空格，+代表出现一次或多次。
```

# 3. sed同时替换多个变量
通过`-e`指定多个替换命令
```shell
 sed -e 's/11/22/g' -e 's/33/44/g' my.txt
```