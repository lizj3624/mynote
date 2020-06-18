[govendor](https://github.com/kardianos/govendor) 是一个基于 vendor 机制实现的 Go 包依赖管理命令行工具。与原生 vendor 无侵入性融合，也支持从其他依赖管理工具迁移，可以很方便的实现同一个包在不同项目中不同版本、以及无相互侵入的开发和管理。
### vendor 特性
最开始的时候，Go 并没有提供较为妥当的包管理工具。从 1.5 版本开始提供了 vendor 特性，但需要手动设置环境变量 `GO15VENDOREXPERIMENT=1`。

在执行 `go build` 或 `go run` 命令时，会按照以下顺序去查找包：

当前包下的 `vendor` 目录
向上级目录查找，直到找到 `src` 下的 `vendor` 目录
在 `GOROOT` 目录下查找
在 `GOPATH` 下面查找依赖包
在发布 `1.6` 版本时，该环境变量的值已经默认设置为 `1` 了，该值可以使用 `go env` 命令查看。

在发布 `1.7` 版本时，已去掉该环境变量，默认开启 `vendor` 特性。
### vendor 使用建议
- 一个库工程（不包含 `main` 的 `package`）不应该在自己的版本控制中存储外部的包在 `vendor` 目录中，除非有特殊原因并且知道为什么要这么做。

- 在一个应用中，（包含 `main` 的 `package`），建议只有一个 `vendor` 目录，且在代码库一级目录。

### govendor 简介
`govendor` 是一个基于 `vendor` 目录机制的包管理工具。

- 支持从项目源码中分析出依赖的包，并从 `$GOPATH` 复制到项目的 `vendor` 目录下
- 支持包的指定版本，并用 `vendor/vendor.json` 进行包和版本管理，这点与 `PHP` 的 `Composer` 类似
- 支持用 `govendor add/update` 命令从 `$GOPATH` 中复制依赖包
- 如果忽略了 `vendor/*/` 文件，可用 `govendor sync` 恢复依赖包
- 可直接用 `govendor fetch` 添加或更新依赖包
- 可用 `govendor migrate` 从其他 `vendor` 包管理工具中一键迁移到`govendor`
- 支持 Linux，macOS，Windows，甚至现有所有操作系统
- 支持 `Git、Hg、SVN，BZR`（必须指定一个路径）

### govendor 使用
要求： - 项目必须在 `$GOPATH/src` 目录下 - 如果 `Go` 版本为 `1.5`，则必须手动设置环境变量 `set GO15VENDOREXPERIMENT=1`   
- 安装
```golang
go get -u github.com/kardianos/govendor
```
为了方便快捷使用 `govendor`，建议将 `$GOPATH/bin` 添加到 `PATH` 中。`Linux/macOS` 如下设置：
```shell
export PATH="$GOPATH/bin:$PATH"
```

- 初始化
在项目根目录下执行以下命令进行 vendor 初始化：
```shell
govendor init
```
项目根目录下即会自动生成 `vendor` 目录和 `vendor.json` 文件。此时 `vendor.json` 文件内容为：
```json
{
	"comment": "",
	"ignore": "test",
	"package": [],
	"rootPath": "govendor-example"
}
```
- 常用命令
1) 将已被引用且在 `$GOPATH` 下的所有包复制到 `vendor` 目录    
```shell
govendor add +external
```
2) 仅从 `$GOPATH` 中复制指定包
```shell
govendor add gopkg.in/yaml.v2
```
3) 列出代码中所有被引用到的包及其状态    
```shell
govendor list
 e  github.com/gin-contrib/sse
 e  github.com/gin-gonic/gin
 e  github.com/gin-gonic/gin/binding
 e  github.com/gin-gonic/gin/internal/json
 e  github.com/gin-gonic/gin/render
 e  github.com/golang/protobuf/proto
 e  github.com/mattn/go-isatty
 e  github.com/ugorji/go/codec
 e  gopkg.in/go-playground/validator.v8
 e  gopkg.in/yaml.v2
pl  govendor-example
  m github.com/json-iterator/go
  m golang.org/x/sys/unix
```
4) 列出一个包被哪些包引用
```shell
govendor list -v fmt
 s  fmt
    ├──  e  github.com/gin-contrib/sse
    ├──  e  github.com/gin-gonic/gin
    ├──  e  github.com/gin-gonic/gin/render
    ├──  e  github.com/golang/protobuf/proto
    ├──  e  github.com/ugorji/go/codec
    ├──  e  gopkg.in/go-playground/validator.v8
    ├──  e  gopkg.in/yaml.v2
    └── pl  govendor-example
```
5) 从远程仓库添加或更新某个包(不会在 `$GOPATH` 也存一份)
```shell
govendor fetch golang.org/x/net/context
```
6) 安装指定版本的包
```shell
govendor fetch golang.org/x/net/context@a4bbce9fcae005b22ae5443f6af064d80a6f5a55
govendor fetch golang.org/x/net/context@v1   # Get latest v1.*.* tag or branch.
govendor fetch golang.org/x/net/context@=v1  # Get the tag or branch named "v1".
```
7) 只格式化项目自身代码(`vendor` 目录下的不变动)
```shell
govendor fmt +local
```
8) 只构建编译项目内部的包
```shell
govendor install +local
```
9) 只测试项目内部的测试案例
```shell
govendor test +local
```
10) 构建所有`vendor`包
```shell
govendor install +vendor,^program
```
11) 拉取所有依赖的包到`vendor`目录(包括`$GOPATH`存在或不存在的包)
```shell
govendor fetch +out
```
12) 包已在`vendor`目录，但想从`$GOPATH`更新
```shell
govendor update +vendor
```

13) 已修改了`$GOPATH`里的某个包，现在想将已修改且未提交的包更新到`vendor`
```shell
govendor update -uncommitted <updated-package-import-path>
```

14) `Fork`了某个包，但尚未合并，该如何引用到最新的代码包
```shell
govendor fetch github.com/normal/pkg::github.com/myfork/pkg
```
此时将从 `myfork` 拉取代码，而不是 `normal`    
15) `vendor.json` 中记录了依赖包信息，该如何拉取更新
```shell
govendor sync
```

### govendor 子命令
各子命令详细用法可通过 `govendor COMMAND -h` 或阅读 `github.com/kardianos/govendor/context` 查看源码包如何实现的。

子命令 | 功能
---|---
init   |	创建 vendor 目录和 vendor.json 文件
list   |	列出&过滤依赖包及其状态
add	   |    从$GOPATH 复制包到项目 vendor 目录
update |	从 $GOPATH 更新依赖包到项目 vendor 目录
remove |	从 vendor 目录移除依赖的包
status |	列出所有缺失、过期和修改过的包
fetch  |	从远程仓库添加或更新包到项目 vendor 目录(不会存储到 $GOPATH)
sync   |	根据 vendor.json 拉取相匹配的包到 vendor 目录
migrate |	从其他基于 vendor 实现的包管理工具中一键迁移
get	   |    与 go get 类似，将包下载到 $GOPATH，再将依赖包复制到 vendor 目录
license |	列出所有依赖包的 LICENSE
shell |	可一次性运行多个 govendor 命令

### govendor 状态参数

状态 | 缩写 | 含义
---|---|---
+local | l | 本地包，即项目内部编写的包
+external | e | 外部包，即在 GOPATH 中、却不在项目 vendor 目录
+vendor	| v | 已在 vendor 目录下的包
+std | s | 标准库里的包
+excluded | x | 明确被排除的外部包
+unused | u | 未使用的包，即在 vendor 目录下，但项目中并未引用到
+missing | m | 被引用了但却找不到的包
+program | p | 主程序包，即可被编译为执行文件的包
+outside |	| 相当于状态为 +external +missing
+all | | 所有包
支持状态参数的子命令有：`list、add、update、remove、fetch`

### Go modules
普大喜奔的是，从 Go 1.11 版本开始，官方已内置了更为强大的 Go modules 来一统多年来 Go 包依赖管理混乱的局面(Go 官方之前推出的 dep 工具也几乎胎死腹中)，并且将在 1.13 版本中正式默认开启。

目前已受到社区的看好和强烈推荐，建议新项目采用 Go modules。

### 参考
[原文链接](https://shockerli.net/post/go-package-manage-tool-govendor/)