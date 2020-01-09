#### Vim一些资料

[简明Vim练级攻略](https://coolshell.cn/articles/5426.html)

![Vim键盘图](https://github.com/lizj3624/mynote/blob/master/vim/pictures/vim键盘图.png)

#### Vim的分屏功能

##### Vim启动时分屏

* 使用大写的字母`O`参数来垂直分屏。

    ```shell
    vim -On file1 file2 ...
    ```

* 使用小写的字母`o`参数来水平分屏

    ```shell
    vim -on file1 file2 ...
    ```

> **注释:** n是数字，表示分成几个屏。

##### Vim打开后分屏

* 上下分割，并打开一个新的文件。

    ```shell
    :sp filename
    ```

* 上下分割，并打开一个新的文件。

    ```shell
    :vsp filename
    ```

##### 光标在窗口间游走

Vim中的光标键是h, j, k, l，要在各个屏间切换，只需要先按一下Ctrl+W

* 把光标移到右边的屏。

```
Ctrl+W l
```

* 把光标移到左边的屏中。

```
Ctrl+W h
```

* 把光标移到上边的屏中。

```
Ctrl+W k
```

* 把光标移到下边的屏中。

```
Ctrl+W j
```

* 把光标移到下一个的屏中。

```shell
Ctrl+W w
```

##### 屏幕尺寸

下面是改变尺寸的一些操作，主要是高度，对于宽度你可以使用`[Ctrl+W <]`或是`[Ctrl+W >]`，但这可能需要最新的版本才支持。

* 让所有的屏都有一样的高度。

```
Ctrl+W =
```

* 增加高度。

```
Ctrl+W +
```

* 减少高度。

```
Ctrl+W -
```

