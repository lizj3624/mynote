# git汇总
    [Git 的奇技淫巧](https://mp.weixin.qq.com/s/K5Xcpf1RytqrjjxwWstvdg)

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

#### 更新submodule的地址

```shell
git submodule update --remote liba
git config -f .gitmodules submodule.liba.branch dev
git submodule update --remote
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

#### 修改远程仓库名称 

```shell
git remote rename origin destination
```

#### 删除远程仓库名称

```shell
git remote rm destination
```

### git修改commit后提交说明

```shell
git commit --amend -m "提交说明"
```

不过在git中，其`commit`提供了一个`--amend`参数，可以修改最后一次提交的信息。但是如果你已经`push`过了，那么其历史最后一次，永远也不能修改了。

### git设置用户和邮箱

```shell
git config --global user.name "jack lee"
git config --global user.email "jack@gmail.com"

#如果你只需要修改最新的 commit ，直接使用：
git commit --amend --author="Author Name <email@address.com>"

#如果你已经修改了 git config 中的用户名和邮箱，也可以使用
git commit --amend --reset-author --no-edit
```

### git revert 和 git reset区别

`git revert`撤销 某次操作，此次操作之前和之后的`commit`和`history`都会保留，并且把这次撤销
作为一次最新的提交

```shell
git revert HEAD         #撤销前一次 commit
git revert HEAD^        #撤销前前一次 commit
git revert commitid     #撤销指定的版本，撤销也会作为一次提交进行保存。
```

`git revert`是提交一个新的版本，将需要`revert`的版本的内容再反向修改回去，
版本会递增，不影响之前提交的内容。

* `git revert` 和 `git reset`的区别 
    1. `git revert`是用一次新的`commit`来回滚之前的`commit`，`git reset`是直接删除指定的`commit`。 
    2. 在回滚这一操作上看，效果差不多。但是在日后继续`merge`以前的老版本时有区别。因为git revert是用一次逆向的`commit`“中和”之前的提交，因此日后合并老的`branch`时，导致这部分改变不会再次出现，但是`git reset`是之间把某些`commit`在某个`branch上`删除，因而和老的`branch`再次`merge`时，这些被回滚的`commit`应该还会被引入。 
    3. `git reset` 是把HEAD向后移动了一下，而`git revert`是HEAD继续前进，只是新的`commit`的内容和要`revert`的内容正好相反，能够抵消要被`revert`的内容。

### git获取某次commit的指定信息（作者，时间，message等）

* 获取某个commit的作者

    ```shell
    $ git log --pretty=format:“%an” b29b8b608b4d00f85b5d08663120b286ea657b4a -1
    “liurizhou”
    ```

* 获取某个commit的时间

    ```shell
    git log --pretty=format:“%cd” b29b8b608b4d00f85b5d08663120b286ea657b4a -1
    “Wed Apr 3 10:12:33 2019 +0800”
    ```
* 显示最新的3条记录
    ```shell
    git log -3 --pretty=oneline
    ```

* 获取某个commit的提交message

    ```shell
    $ git log --pretty=format:“%s” b29b8b608b4d00f85b5d08663120b286ea657b4a -1
    “Change the length of the pre label string.”
    ```

* 其中--pretty=format:“%xx”可以指定需要的信息，其常用的选项有

    ```shell
    %H: commit hash
    %h: 缩短的commit hash
    %T: tree hash
    %t: 缩短的 tree hash
    %P: parent hashes
    %p: 缩短的 parent hashes
    %an: 作者名字
    %aN: mailmap的作者名字 (.mailmap对应，详情参照git-shortlog(1)或者git-blame(1))
    %ae: 作者邮箱
    %aE: 作者邮箱 (.mailmap对应，详情参照git-shortlog(1)或者git-blame(1))
    %ad: 日期 (--date= 制定的格式)
    %aD: 日期, RFC2822格式
    %ar: 日期, 相对格式(1 day ago)
    %at: 日期, UNIX timestamp
    %ai: 日期, ISO 8601 格式
    %cn: 提交者名字
    %cN: 提交者名字 (.mailmap对应，详情参照git-shortlog(1)或者git-blame(1))
    %ce: 提交者 email
    %cE: 提交者 email (.mailmap对应，详情参照git-shortlog(1)或者git-blame(1))
    %cd: 提交日期 (--date= 制定的格式)
    %cD: 提交日期, RFC2822格式
    %cr: 提交日期, 相对格式(1 day ago)
    %ct: 提交日期, UNIX timestamp
    %ci: 提交日期, ISO 8601 格式
    %d: ref名称
    %e: encoding
    %s: commit信息标题
    %f: sanitized subject line, suitable for a filename
    %b: commit信息内容
    %N: commit notes
    %gD: reflog selector, e.g., refs/stash@{1}
    %gd: shortened reflog selector, e.g., stash@{1}
    %gs: reflog subject
    %Cred: 切换到红色
    %Cgreen: 切换到绿色
    %Cblue: 切换到蓝色
    %Creset: 重设颜色
    %C(...): 制定颜色, as described in color.branch.* config option
    %m: left, right or boundary mark
    %n: 换行
    %%: a raw %
    %x00: print a byte from a hex code
    %w([[,[,]]]): switch line wrapping, like the -w option of git-shortlog(1).
    ```
* 获取commit id

    ```shell
    # 获取完整commit id
    git rev-parse HEAD

    # 获取short commit id
    git rev-parse --short HEAD
    ```
* git status乱码
   ```shell
   git config --global core.quotepath false
   ```

* git pull/push每次都要输入用户名密码的解决办法
  ```shell
  git config --global credential.helper store
  ```
