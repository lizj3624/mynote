---
title: "Linux下的cordump设置和使用"
date: 2022-05-30T17:56:33+08:00
tags:
   - Linux 
   - coredump
categories:
   - Linux 
   - coredump
toc: true
---

`coredump`是进程崩溃前的快照，对程序员定位问题非常有用，再次记录一下`coredump`的设置和使用。
### 设置`coredump`
```shell
# 1. 设置core_uses_pid 
echo "1" > /proc/sys/kernel/core_uses_pid

# 2. 设置core_pattern, 进程所属用户必须有目录权限
echo "/myservers/srv/bin/core-%e-%p-%t" > /proc/sys/kernel/core_pattern

# 3. 设置ulimt 
echo "ulimit -c unlimited" >> /etc/profile && source /etc/profile

# 4. 重启服务 
```

### 使用`coredump`

`gdb` + `可执行文件名` + `coredump文件` 即可开始调试`coredump`文件, 通过`b`定位段点。

### 手工产生`coredump`
1. `gcore $pid`

2. `gdb generate-core-file`
