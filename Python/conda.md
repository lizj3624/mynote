## Anaconda

[Anaconda](https://www.anaconda.com/products/enterprise)是一个开源的工具，目前拥有超过六百万的用户。`Anaconda`致力于提供最便捷的方式来使用`Python`进行数据科学计算和机器学习。目前，Anaconda拥有超过250+的数据科学工具包，conda工具包可用于`Windows`，`MacOS`和`Linux`三种平台的虚拟环境管理系统。Anaconda支持当前比较流行的一些人工智能的库，比如`Sklearn`，`TensorFlow`，`Scipy`。

## 添加常用源

由于网络问题，有些时候直接同国外下载库会比较慢，我们可以给`conda`配置国内的镜像源，添加国内的镜像源命令如下：

1. 清华源

```shell
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
# 设置搜索时显示通道地址
conda config --set show_channel_urls yes
```

2. 添加中科院源

```shell
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/
# 设置搜索时显示通道地址
conda config --set show_channel_urls yes
```

查看是否添加成功可是用命令

```shell
conda config --show
```

在 `channels`这个字段这里显示已经添加的源

```shell
channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
```

## conda常用命令

### **升级**

```shell
conda update conda  # 更新 conda
conda update anaconda # 更新 anaconda
conda update anaconda-navigator    #update最新版本的anaconda-navigator  
conda update python # 更新 python
```

### **管理环境**

```shell
conda env list  #显示所有的虚拟环境
conda create --name fulade python=3.7 # 创建一个名为 fulade 环境，指定Python版本是3.7
activate fulade  # 激活名为 fulade 的环境 (Windows 使用)
source activate fulade  # 激活名为 fulade 的环境 (Linux & Mac使用用)
deactivate fulade   #关闭名为 fulade的环境( Windows使用)
source deactivate fulade  # 关闭名为 fulade的环境(Linux & Mac使用）
conda remove --name fulade --all # 删除一个名为 fulade 的环境
conda create --name newname --clone oldname # 克隆oldname环境为newname环境
```

### **package管理**

```shell
conda list  #查看当前环境下已安装的package
conda search numpy # 查找名为 numpy 的信息 package 的信息
conda install numpy  # 安装名字为 fulade 的package 安装命令使用-n指定环境 --channel指定源地址
conda install -n fulade numpy  # 在fulade的环境中 安装名字为 fulade 的package
conda install --channel https://conda.anaconda.org/anaconda tensorflow=1.8.0  # 使用地址 https://conda.anaconda.org/anaconda 来安装tensorflow
conda update numpy   #更新numpy package
conda uninstall numpy   #卸载numpy package
```

### **清理conda**

```shell
conda clean -p      //删除没有用的包
conda clean -t      //删除tar包
conda clean -y --all //删除所有的安装包及cache
```

## conda与pip

利用`conda install`与`pip install`命令来安装各种包的过程中，想必你也对两者之间的区别很疑惑，下面我就总结一下我搜集到的相关解答。
简而言之，pip是python包的通用管理器，而conda是一个与语言无关的跨平台环境管理器。对我们而言，最显着的区别可能是这样的：pip在任何环境中安装python包，conda安装在conda环境中装任何包。因此往往conda list的数量会大于pip list。
要注意的是，如果使用conda安装多个环境时，对于同一个包只需要安装一次，有conda集中进行管理。
但是如果使用pip，因为每个环境安装使用的pip在不同的路径下，故会重复安装，而包会从缓存中取。
总的来说，我推荐尽早安装anaconda并且使用conda来管理python的各种包。

## 引用

* [conda-command](https://docs.conda.io/projects/conda/en/latest/commands.html)

