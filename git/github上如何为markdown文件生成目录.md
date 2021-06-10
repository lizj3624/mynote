## 写在前面

熟悉markdown都知道可以使用[TOC]自动生成markdown文件的标题目录，比如在typora，vscode(需要插件)等本地编辑器中，或者在CSDN等网页编辑器中，但是github却不支持[TOC]标签，至于为什么不支持感兴趣的可以深入搜索，而相应的解决方法之一就是为md文件自动生成适用于github的目录。

## 网页生成

有没有这样自动生成目录的操作呢？答案是有的，其中之一就是在线的generator，比如[GitHub Wiki TOC generator](https://link.zhihu.com/?target=https%3A//ecotrust-canada.github.io/markdown-toc/)。但是经本人实测并不好用且麻烦。

## 使用toc工具

网页生成的方法并不好用，使用toc工具，比如gh-md-toc这个神器，如何安装可参考这篇博文[快速生成Github README.md的目录](https://link.zhihu.com/?target=https%3A//www.jianshu.com/p/302abe331dcb)。安装后在终端输入命令就可以在终端里生成目录，但是也比较麻烦，因为是在终端生成的，复制粘贴到md文件里比较麻烦，还要注意格式对齐的问题。

## 最简单的markdown插件

笔者目前了解到的最最最简单的莫过于VSCode中的Markdown All in One 插件了，安装后点开md文件，然后快捷键`CTRL(CMD)+SHIFT+P`，输入`Markdown All in One: Create Table of Contents`回车即可，如下：

![img](https://pic1.zhimg.com/80/v2-c49d23f77f29f0ac10317810ccff1db4_1440w.jpg)

![img](https://pic4.zhimg.com/80/v2-fce272493be2a38ece7880e93eb9364f_1440w.jpg)

![img](https://pic3.zhimg.com/80/v2-6fd22728ecdef2e82b80ecb3a6e061ae_1440w.jpg)