

#### 前言

在传统的 C/C++ 等项目构建时，通常会采用 make 系统使用 Makefile 文件来进行整个项目的编译构建，通过 Makefile 中指定的编译所依赖的规则使得程序的构建非常简单，并且在复杂项目中可以避免由于少部分源码修改而造成的很多不必要的重编译。但是它仍然不够好，因为其大而且复杂，有时候我们并不需要 make 那么强大的功能，相反我们需要更灵活，速度更快的编译工具。Ninja 作为一个新型的编译工具，小巧而又高效，它就是为此而生。

**这篇文章介绍 Ninja 的安装以及如何使用 Ninja 来构建项目**

首先，我们需要安装 Ninja，只需要去官网下载一个 release 的二进制版本，放在系统目录（比如 /usr/bin）中就可以了，非常的简单。另外，现在大多数 Linux 发行版都有自己的包管理工具，直接使用包管理工具来下载也很简单。

下面简单介绍下通过编译 Ninja 源码的方式来安装
 首先，确保已经安装了这些依赖：g++，graphviz，gtest，git，re2c 和 python2.7+。

###### 获取源码

```shell
$ git clone git://github.com/ninja-build/ninja.git && cd ninja
$ git checkout release
$ cat README

$ ls
COPYING  HACKING.md  README  RELEASING  bootstrap.py  configure.py  doc/  misc/  src/
```

我们可以去 HACKING.md 中查看更多信息。

###### 编译

一切就绪之后，执行下列命令来编译 ninja



```shell
$ ./configure.py --bootstrap
```

上述命令会在当前目录下生成一个叫 ninja （Windows 下是 ninja.exe）的可执行文件，然后我们把这个文件拷到系统目录（比如 /usr/bin）就完成安装了。

###### 编译过程解析

实际上 ninja 本身也是通过 ninja 系统来编译完成的。
 具体过程就是：执行 `./configure.py --bootstrap` 之后先编译源码（生成一个 a.out），然后在当前目录生成一个 ninja.build（这个文件类似于 make 工具的 Makefile，语法和规则非常类似）。然后再根据这个 ninja.build 来重新编译生成可执行文件 ninja，在 ninja 根据 ninja.build 来编译时会自动创建一个 build 目录用于存放编译过程中的临时文件，比如 *.o 等。

执行`./configure.py` 时还可以指定其他选项：

**--bootstrap**　　　 bootstrap a ninja binary from nothing
 **--verbose**　　　　enable verbose build
 **--platform** 　　　  choose known platforms
 **--host** 　　　　　 choose host known_platforms
 **--debug** 　　　　  enable debugging extras
 **--profile** 　　　　  enable profiling
 **--with-gtest**
 **--with-python** 　　use EXE as the Python interpreter
 **--force-pselect** 　　ppoll() is used by default where available

可以通过 `./configure.py -h` 可以查看更多帮助。
 如果我们想要开启 Ninja 的其他特性（比如：Bash completion， Emacs 和 Vim 编辑模式等），编译完成之后，我们需要把 /misc 目录中的文件拷贝到合适的位置。

###### 测试

现在，我们可以测试一下 ninja 是否成功安装并且可以使用。

当直接执行 `ninja` 命令是，它会在当前目录下默认寻找 build.ninja 文件来进行编译。
 ninja 的语法格式是：



```shell
$ ninja [options] TARGETs
```

上述 options 如果没有则可以省略。比如，直接执行 `./ninja ninja_test` 将会生成可执行文件 ninja_test，然后再执行 ninja_test 就可以看到测试结果。

如下：



```shell
$ ./ninja ninja_test
$ ./ninja_test
[214/226] SubprocessTest.SetWithLotsRaise [ulimit -n] above 1025 (currently 1024) to make this test go
[226/226] ElideMiddle.ElideInTheMiddle
passed
```

或者，我们还可以直接执行 `./ninja all`，这样，ninja 就会执行 ninja.build 中指定的所有目标了。



```shell
$ ./ninja all
[10/10] LINK canon_perftest
```

上述 ninja_test 和  all 都是 ninja.build 中的 build rule，概念类似于 Makefile 中的 target recipe。

测试完成之后，我们就把 ninja 拷贝到一个系统目录中 /usr/bin 来完成整个的安装。

**提示：** build.ninja 文件类似于 Makefile，熟悉它的语法规则之后我们也可以手动编写。另外，可以通过 `ninja -f NINJA_FILE` 的方式来指定 .ninja 文件

更多选项

![img](https:////upload-images.jianshu.io/upload_images/1452123-566a607e7a4cb640.png?imageMogr2/auto-orient/strip|imageView2/2/w/563/format/webp)

实际上, ninja 还提供了一个 [Python based generator](https://link.jianshu.com?t=https://github.com/martine/ninja/blob/84986/misc/ninja_syntax.py) ，它实际上是一个 Python 模块 `misc/ninja_syntax.py`，通过它我们可以较方便的生成 build.ninja 文件。比如，在我们的 Python 文件中引入该模块之后，就可以直接通过调用 `ninja.rule(name='foo', command='bar', depfile='$out.d')` 来生成符合 ninja 语法的内容。下面是一个简单例子：

```python
from ninja_syntax import Writer

with open("build.ninja", "w") as buildfile:
    n = Writer(buildfile)

    if platform.is_msvc():
        n.rule("link",
                command="$cxx $in $libs /nologo /link $ldflags /out:$out",
                description="LINK $out")
    else:
        n.rule("link",
                command="$cxx $ldflags -o $out $in $libs",
                description="LINK $out")
```

另外，我们还可以在执行 cmake 时通过 -G 选项指定生成器为 ninja 来生成 build.ninja。
 比如：



```shell
$ cd build 
$ cmake -GNinja ../proj_src_dir
```

#### Ninja 工具集

Ninja 还集成了 graphviz 等一些对开发非常有用的工具，通过执行 `./ninja -t list` 可以查看 ninja 中集成了哪些工具。

下面是一个常见的工具集列表：

```bash
ninja subtools:

browse        # 在浏览器中浏览依赖关系图。（默认会在 8080 端口启动一个基于python的http服务）
clean         # 清除构建生成的文件
commands      # 罗列重新构建制定目标所需的所有命令
deps          # 显示存储在deps日志中的依赖关系
graph         # 为指定目标生成 graphviz dot 文件。
                如 ninja -t graph all |dot -Tpng -ograph.png
query         # 显示一个路径的inputs/outputs
targets       # 通过DAG中rule或depth罗列target
compdb        # dump JSON兼容的数据库到标准输出
recompact     # 重新紧凑化ninja内部数据结构
```

#### 手动编写 .ninja 文件

.ninja 的语法规则跟 Makefile 类似，虽然有许多 [generator 工具](https://link.jianshu.com?t=https://github.com/ninja-build/ninja/wiki/List-of-generators-producing-ninja-build-files) 可以用来自动生成 .ninja 文件，但是在某些场合可能需要手动编写或修改 .ninja 文件，下面做个简单介绍：



```ninja
# VARIABLE: (referenced like $name or alternate ${name})
cflag = -g -Wall -Werror

# RULE:
rule RULE_NAME
    command = gcc $cflags -c $in -o $out
    description = ${out} will be treat as "$out"

# BUILD statement:
build TARGET_NAME: RULE_NAME INPUTS

# PHONE rule:(creating alias)
build ALIAS: phony INPUTS ...

# DEFAULT target statement(cumulative):
default TARGET1 TARGET2
default TARGET3

    $ ninja
    build TARGET1 TARGET2 TARGET3
```

例子： `build.ninja`



```ninja
ninja_required_version = 1.3

#variable
cc = g++
cflags = -Wall

# rule
rule cc
    command = gcc $cflags -c $in -o $out
    description = compile .cc

# build
build foo.o: cc foo.c
```

`.ninja_log` 可以用来指定保存 build 时产生的 log。

##### 子模块 和 include 指令

subninja 指令可以用来引入其他 .ninja 文件，从而引入一个新的 scope。这意味着，子模块中可以引用父模块中的变量。
 比如：

```ninja
subninja obj/content/content_resources.ninja
subninja obj/extensions/extensions_strings.ninja
subninja obj/third_party/expat/expat_nacl.ninja
```

include 指令也是用来引入其他 .ninja 文件，但是不同的是，引入的其他 .ninja 文件会被引入当前 scope，子模块中不可以访问父模块中的变量。

```ninja
include obj/content/content_resource.ninja
include obj/extensions/extensions_strings.ninja
include obj/third_party/expat/expat_nacl.ninja
```

**参考**
 [https://github.com/ninja-build/ninja](https://link.jianshu.com?t=https://github.com/ninja-build/ninja)
 [https://ninja-build.org/manual.html](https://link.jianshu.com?t=https://ninja-build.org/manual.html)

