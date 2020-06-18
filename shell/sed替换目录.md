#### 1、用sed实现路径替换
```shell
sed "s:${SRC_PATH}:${DST_PATH}:g" $filename
```
一般情况下，`sed 's/pattern/pattern/flag'`是用`/`来进行分隔的。但这里的源字符串和替换字符串中，都带有字符`/`，如果再用`/`进行分隔，就会产生冲突。所以改用了`：`进行分隔。

### 2、sed几种替换
```shell
a="one"
b="two"
# 第一种：
eval sed -i ’s/$a/$b/’ filename
# 第二种（推荐）：
sed -i "s/$a/$b/" filename
# 第三种：
sed -i ’s/’$a’/’$b’/’ filename 
# 第四种：
sed -i s/$a/$b/ filename
```
如果变量`a`或者`b`有` -、/` 等字符要用 `\` 进行转义
字符串变量中可以用单引号或者双引号 ，区别`：`双引号支持变量引用、转义符（比如`\n` 换行），单引号不支持
`sed` 命令执行时加 `-i` 参数会把修改应用到源文件上，否则只是屏幕显示