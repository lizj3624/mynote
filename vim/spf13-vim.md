#spf13-vim

* spf13-vim是针对Vim、gVim、MacVim，包含众多Vim插件和相关资源的Vim发行版

* [spf13-vim](https://vim.spf13.com/)

* [spf13-vim github](https://vim.spf13.com/)

* [spf13-vim - Vim编辑器的终极版本](https://www.linuxidc.com/Linux/2018-05/152570.htm)

##Install
### install script

```shell
curl http://j.mp/spf13-vim3 -L -o - | sh

echo colorscheme solarized  >> ~/.vimrc.local
```

### install遇到问题

#### 色块问题

```shell
在~/.vimrc.local添加
autocmd VimEnter * set nospell
```

#### 安装YCM时报错

```shell
./install.sh -clang-completer 找不到"Python.h"
yum install python-dev
```

#### 设置tab为4个空格
```shell
~/.vimrc.loca中设置tab为4个空格失效

解决办法是删除~/.vimviews目录的内容，因此将这一目录删除，并在~/.vimrc.local中添加你的配置，重新打开vim即可更新vim的配置信息。
```
### spf13-vim常用快捷键、功能

[常用快捷键](https://blog.csdn.net/BjarneCpp/article/details/80608706)
