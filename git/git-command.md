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

