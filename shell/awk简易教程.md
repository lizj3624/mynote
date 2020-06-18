#### 相关教程
- [The GNU Awk User’s Guide](http://www.gnu.org/software/gawk/manual/gawk.html)
- [awk简易教程](https://coolshell.cn/articles/9070.html )

#### 1、awk 打印从某一列到最后一列的内容
```shell
awk -F‘ ’ '{for (i=2;i<=NF;i++)printf("%s ", $i);print ""}'
```
```shell
awk -F‘ ’  #以空格为分隔符

for (i=2;i<=NF;i++) printf("%s ",$i)  #从第二列开始到最后，注意%s 后面有空格。

print “” #打印组合
```
#### 2、awk打印输出每一列，并在每列中间加入"|"
```shell
cat $file |awk 'BEGIN{str=""} {for(i=1;i<=NF;i++){if(i==1) str=$1; else {str=str"""|";str=str""$i}} print str; str=""}'>$1.new
```

#### awk内置变量
- ARGC   命令行参数个数
- ARGV               命令行参数排列
- ENVIRON          支持队列中系统环境变量的使用
- FILENAME         awk浏览的文件名
- FNR                  浏览文件的记录数
- FS                    设置输入域分隔符，等价于命令行 -F选项
- NF                    浏览记录的域的个数
- NR                    已读的记录数
- OFS                  输出域分隔符
- ORS                  输出记录分隔符
- RS                    控制记录分隔符
#### 常用函数
awk内置函数，主要分以下3种类似：算数函数、字符串函数、其它一般函数、时间函数

函数 | 说明
------|-------
gsub( Ere, Repl, [ In ] )	| 除了正则表达式所有具体值被替代这点，它和 sub 函数完全一样地执行，。
sub( Ere, Repl, [ In ] ) |	用 Repl 参数指定的字符串替换 In 参数指定的字符串中的由 Ere 参数指定的扩展正则表达式的第一个具体值。sub 函数返回替换的数量。出现在 Repl 参数指定的字符串中的 &（和符号）由 In 参数指定的与 Ere 参数的指定的扩展正则表达式匹配的字符串替换。如果未指定 In 参数，缺省值是整个记录（$0 记录变量）。
index( String1, String2 ) |	在由 String1 参数指定的字符串（其中有出现 String2 指定的参数）中，返回位置，从 1 开始编号。如果 String2 参数不在 String1 参数中出现，则返回 0（零）。
length [(String)] |	返回 String 参数指定的字符串的长度（字符形式）。如果未给出 String 参数，则返回整个记录的长度（$0 记录变量）。
blength [(String)] |	返回 String 参数指定的字符串的长度（以字节为单位）。如果未给出 String 参数，则返回整个记录的长度（$0 记录变量）。
substr( String, M, [ N ] ) |	返回具有 N 参数指定的字符数量子串。子串从 String 参数指定的字符串取得，其字符以 M 参数指定的位置开始。M 参数指定为将 String 参数中的第一个字符作为编号 1。如果未指定 N 参数，则子串的长度将是 M 参数指定的位置到 String 参数的末尾 的长度。
match( String, Ere ) |	在 String 参数指定的字符串（Ere 参数指定的扩展正则表达式出现在其中）中返回位置（字符形式），从 1 开始编号，或如果 Ere 参数不出现，则返回 0（零）。RSTART 特殊变量设置为返回值。RLENGTH 特殊变量设置为匹配的字符串的长度，或如果未找到任何匹配，则设置为 -1（负一）。
split( String, A, [Ere] ) |	将 String 参数指定的参数分割为数组元素 A[1], A[2], . . ., A[n]，并返回 n 变量的值。此分隔可以通过 Ere 参数指定的扩展正则表达式进行，或用当前字段分隔符（FS 特殊变量）来进行（如果没有给出 Ere 参数）。除非上下文指明特定的元素还应具有一个数字值，否则 A 数组中的元素用字符串值来创建。
tolower( String )	| 返回 String 参数指定的字符串，字符串中每个大写字符将更改为小写。大写和小写的映射由当前语言环境的 LC_CTYPE 范畴定义。
toupper( String )	| 返回 String 参数指定的字符串，字符串中每个小写字符将更改为大写。大写和小写的映射由当前语言环境的 LC_CTYPE 范畴定义。
sprintf(Format, Expr, Expr, . . . ) |	根据 Format 参数指定的 printf 子例程格式字符串来格式化 Expr 参数指定的表达式并返回最后生成的字符串。

#### print和printf
awk中同时提供了print和printf两种打印输出的函数。
- 其中print函数的参数可以是变量、数值或者字符串。字符串必须用双引号引用，参数用逗号分隔。如果没有逗号，参数就串联在一起而无法区分。这里，逗号的作用与输出文件的分隔符的作用是一样的，只是后者是空格而已。
- printf函数，其用法和c语言中printf基本相似,可以格式化字符串,输出复杂时，printf更加好用，代码更易懂。
```shell
awk  -F ':'  '{printf("filename:%10s,linenumber:%s,columns:%s,linecontent:%s\n",FILENAME,NR,NF,$0)}' /etc/passwd
```

#### 条件语句
awk中的循环语句同样借鉴于C语言，支持while、do/while、for、break、continue，这些关键字的语义和C语言中的语义完全相同。

#### 数组
因为awk中数组的下标可以是数字和字母，数组的下标通常被称为关键字(key)。值和关键字都存储在内部的一张针对key/value应用hash的表格里。由于hash不是顺序存储，因此在显示数组内容时会发现，它们并不是按照你预料的顺序显示出来的。数组和变量一样，都是在使用时自动创建的，awk也同样会自动判断其存储的是数字还是字符串。一般而言，awk中的数组用来从记录中收集信息，可以用于计算总和、统计单词以及跟踪模板被匹配的次数等等。
```shell
awk -F ':' 'BEGIN {count=0;} {name[count] = $1;count++;}; END{for (i = 0; i < NR; i++) print i, name[i]}' /etc/passwd
```

#### 获取外部变量
- 格式如：awk –v 变量名=变量值 [–v 变量2=值2 …] 'BEGIN{action}'
```shell
     test='awk code' 
     echo | awk -v test="$test" '{print test}'   
```
- 获得环境变量 
awk内置变量 ENVIRON,就可以直接获得环境变量。它是一个字典数组。环境变量名 就是它的键值
```shell
      awk  'BEGIN{for (i in ENVIRON) {print i"="ENVIRON[i];}}'
```      