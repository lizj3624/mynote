记录一下VirtualBox下扩展CentOS 7的vdi虚拟磁盘容量。目前是16G，我想扩展为32G

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818053144437.png)

# 一、调整vdi文件容量

0.**调整容量前，先关闭虚拟机**

1.启动CMD，进入VirtualBox的安装目录，比如：E:\Program File\VirtualBox

```
cd E:\Program File\VirtualBox
```

执行命令，查看目前挂载的虚拟机硬盘信息

```
VBoxManage list hdds
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818053439394.png)

记录下上图红框中的第一行UUID数据，比如：a10ee208-adbd-4819-9396-4a3d881c4d42

执行命令修改其大小，比如我要修改为32G=32*1024=32768MB，格式如：

```
#VBoxManage modifyhd a10ee208-adbd-4819-9396-4a3d881c4d42 --resize 32768
VBoxManage modifyhd 虚拟机硬盘UUID --resize 大小（单位MB）
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818053928738.png)

再次查看挂载的虚拟机硬盘信息

```
VBoxManage list hdds
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818054102250.png)

容量已变更为32768MB（32GB）

# 二、在虚拟机内分配空间

### 1.查询容量，使用 fdisk -l 命令

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818061312847.png)

如上图所示，已分配16G，总容量 – 已分配 = 未分配容量，既为第一步中我们调整的容量。

这部分空间需要分配挂载之后才可以使用。

### 2.开始分区

因为此虚拟机只有一块虚拟硬盘，即 /dev/sda，所以首先需要对此块硬盘的未分配空间进行分区操作，命令：

```
fdisk /dev/sda
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818064545437.png)

分区完毕，输入 w 保存分区表

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818062222853.png)

此时会出现提示：

```
WARNING:Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```

大致意思就是**设备忙，需要重启虚拟机以便分区表生效**。好的，那就重启。

### 3.格式化分区

重启虚拟机后，再次执行 fdisk -l 命令查看，发现多出一个分区

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818062615284.png)

格式化该分区为ext4格式

```
mkfs.ext4 /dev/sda4
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818062818778.png)

# 三、挂载&扩展

此时可以选择是**直接挂载该分区**，还是**扩展已有分区**

### 1).选择直接挂载

可以使用mount命令，将刚刚格式化的分区挂载到某一路径下，如挂载到/home/extend

```
mkdir /home/ext
mount /dev/sda4 /home/ext
```

再修改/etc/fstab，尾部添加一行

```
/dev/sda4 /home/ext ext4 defaults 0 1
```

重启即可自动挂载该分区

### 2).选择扩展已有分区

执行df -h 查看已挂载分区及其挂载路径

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818063412940.png)

比如我要扩展上图红框的根目录 /

首先查看卷组的信息，记录组名称

```
vgdisplay
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818063724972.png)

创建新的物理分区（ /dev/sda4即为上述步骤中扩展的分区 ）

```
pvcreate /dev/sda4
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818064813129.png)

执行 vgextend 扩展命令

```
#格式：vgextend 组名称 扩展分区
vgextend centos /dev/sda4
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818064838659.png)

执行 lvdisplay 指令，显示逻辑卷属性，并记录根分区路径

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818065113491.png)

执行扩展命令

```
lvextend /dev/centos/root /dev/sda4
```

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818065204963.png)

如上图，容量已成功扩展

刷新一下分区容量

```
xfs_growfs /dev/centos/root
```

再执行df -h

![img](https://bugxia.com/wp-content/uploads/2018/08/20180818065420239.png)

扩展容量已成功添加至 / 根目录