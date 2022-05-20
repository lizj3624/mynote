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

# 4. 根据key替换value
```json
{
    "wafCoreTimeout": 5
}
```

```shell
sed -i '/wafCoreTimeout/s/5/10/g' /export/servers/jfe/conf/localconfs/waf.json
```

# 5. 交换行,合并行,删除行
```shell
# sed方法
# N 追加下一个输入行到模式空间，用了两次把当前行的后两行都追加到了模式空间，即多行模式空间。
# 让后用s将\n换行符替换成空格。
# 最后的g是全局替换即替换所有的\n，若不加g表示只替换第一个。

sed 'N;N;s/\n/ /g'  test

# awk方法：
# NR当前行记录数，ORS输出记录分隔符。'ORS=NR%3?" ":"\n" 为三目运算，即若NR对3取莫为0，ORS=“\n”,不为0，ORS=“”。
$ awk 'ORS=NR%3?" ":"\n"{print}' test1

$ cat xai
303728
303778
304175
304176
304261
304470

## 合并上下两行
### sed用法
sed 'N;s/\n/ :/' xai 
303728 :303778
304175 :304176
304261 :304470

### awk用法
$ awk '{if(NR%2==0){printf $0 "\n"}else{printf "%s：",$0}}' xai 
303728：303778
304175：304176
304261：304470

## 合并匹配模式及其下一行
$ sed '/304175/{N;s/\n/\t/}' xai
303728
303778
304175  304176
304261
304470

## 合并所有行
$ sed ':a;N;s/\n/\t/;ba;' xai
303728  303778  304175  304176  304261  304470

## 已知行号时交换两行, 这里是交换1,4行.当然你可以根据自己需要修改
$ cat xai
303728
303778
304175
304176
304261
304470

$ for(( i=1;i<=4;i++ )); do  case $i in 1) sed -n 4p xai;; 4) sed -n 1p xai;; *) sed -n ${i}p xai;; esac; done               
304176
303778
304175
303728

## 删除空行
sed '/^$/d' test2

## 删除多个空行为一个空行
sed '/^$/{N;/^\n*$/D}' test
```
