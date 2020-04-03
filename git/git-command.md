# 常用的git命令

### git子模块

#### 在原有项目增加一个子项目

```shell
 git submodule add https://github.com/chaconinc/DbConnector
```

#### 初始化本地子项目

```shell
git submodule init
```

#### 更新本地子项目

```shell
git submodule update
```

#### clone带有子项目的项目

```shell
$ git clone --recursive https://github.com/chaconinc/MainProject
```

### git添加修改远端仓库地址

#### 添加远端仓库地址

```sh
git remote add origin(可修改) branch_Name(为空时默认为master) url
```

#### 修改远端仓库地址

```shell
git remote set-url origin url
```

### git修改commit后提交说明

```shell
git commit --amend "提交说明"
```

不过在git中，其`commit`提供了一个`--amend`参数，可以修改最后一次提交的信息。但是如果你已经`push`过了，那么其历史最后一次，永远也不能修改了。

### git设置用户和邮箱

```shell
git config --global user.name "jack lee"
git config --global user.email "jack@gmail.com"
```

