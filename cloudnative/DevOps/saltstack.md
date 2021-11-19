## salt简介

SaltStack是一个服务器基础架构集中化管理平台，具备配置管理、远程执行、监控等功能，基于Python语言实现，结合轻量级消息队列（ZeroMQ）与Python第三方模块（Pyzmq、PyCrypto、Pyjinjia2、python-msgpack和PyYAML等）构建。

通过部署SaltStack，我们可以在成千万台服务器上做到批量执行命令，根据不同业务进行配置集中化管理、分发文件、采集服务器数据、操作系统基础及软件包管理等，SaltStack是运维人员提高工作效率、规范业务配置与操作的利器。

##  

### salt基本原理

SaltStack 采用 C/S模式，server端就是salt的master，client端就是minion，minion与master之间通过ZeroMQ消息队列通信

minion上线后先与master端联系，把自己的pub key发过去，这时master端通过salt-key -L命令就会看到minion的key，接受该minion-key后，也就是master与minion已经互信

master可以发送任何指令让minion执行了，salt有很多可执行模块，比如说cmd模块，在安装minion的时候已经自带了，它们通常位于你的python库中，`locate salt | grep /usr/` 可以看到salt自带的所有东西。

这些模块是python写成的文件，里面会有好多函数，如cmd.run，当我们执行`salt '*' cmd.run 'uptime'`的时候，master下发任务匹配到的minion上去，minion执行模块函数，并返回结果。master监听4505和4506端口，4505对应的是ZMQ的PUB system，用来发送消息，4506对应的是REP system是来接受消息的。

具体步骤如下

1. Salt stack的Master与Minion之间通过ZeroMq进行消息传递，使用了ZeroMq的发布-订阅模式，连接方式包括tcp，ipc
2. salt命令，将`cmd.run ls`命令从`salt.client.LocalClient.cmd_cli`发布到master，获取一个Jodid，根据jobid获取命令执行结果。
3. master接收到命令后，将要执行的命令发送给客户端minion。
4. minion从消息总线上接收到要处理的命令，交给`minion._handle_aes`处理
5. `minion._handle_aes`发起一个本地线程调用cmdmod执行ls命令。线程执行完ls后，调用`minion._return_pub`方法，将执行结果通过消息总线返回给master
6. master接收到客户端返回的结果，调用`master._handle_aes`方法，将结果写的文件中
7. `salt.client.LocalClient.cmd_cli`通过轮询获取Job执行结果，将结果输出到终端



### 安装salt

```shell
yum install salt-master
yum install salt-minion
yum install salt-ssh
yum install salt-syndic
yum install salt-cloud
```

#### 配置salt master

```yaml
#指定master，冒号后有一个空格
master: 192.168.2.22
user: root

#-------以下为可选--------------
# salt运行的用户，影响到salt的执行权限
user: root
#s alt的运行线程，开的线程越多一般处理的速度越快，但一般不要超过CPU的个数
worker_threads: 10
# master的管理端口
publish_port : 4505
# master跟minion的通讯端口，用于文件服务，认证，接受返回结果等
ret_port : 4506
# 如果这个master运行的salt-syndic连接到了一个更高层级的master,那么这个参数需要配置成连接到的这个高层级master的监听端口
syndic_master_port : 4506
# 指定pid文件位置
pidfile: /var/run/salt-master.pid
# saltstack 可以控制的文件系统的开始位置
root_dir: /
# 日志文件地址
log_file: /var/log/salt_master.log
# 分组设置
nodegroups:
  group_all: '*'
# salt state执行时候的根目录
file_roots:
  base:
    - /srv/salt/
# 设置pillar 的根目录
pillar_roots:
  base:
    - /srv/pillar
```

```shell
systemctl start salt-master
systemctl enable salt-master
```

#### 配置minion

```yaml
#指定master，冒号后有一个空格
master: 192.168.2.22
id: minion-01
user: root

#-------以下为可选--------------
# minion的识别ID，可以是IP，域名，或是可以通过DNS解析的字符串
id: 192.168.0.100
# salt运行的用户权限
user: root
# master的识别ID，可以是IP，域名，或是可以通过DNS解析的字符串
master : 192.168.0.100
# master通讯端口
master_port: 4506
# 备份模式，minion是本地备份，当进行文件管理时的文件备份模式
backup_mode: minion
# 执行salt-call时候的输出方式
output: nested 
# minion等待master接受认证的时间
acceptance_wait_time: 10
# 失败重连次数，0表示无限次，非零会不断尝试到设置值后停止尝试
acceptance_wait_time_max: 0
# 重新认证延迟时间，可以避免因为master的key改变导致minion需要重新认证的syn风暴
random_reauth_delay: 60
# 日志文件位置
log_file: /var/logs/salt_minion.log
# 文件路径基本位置
file_roots:
  base:
    - /etc/salt/minion/file
# pillar基本位置
pillar_roots:
  base:
    - /data/salt/minion/pillar
```

```shell
systemctl start salt-master
systemctl enable salt-master
```

#### 添加key

```shell
[root@master salt]# salt-key 
Accepted Keys:
Denied Keys:
Unaccepted Keys:   #可看到 minion已经检测到，没有认证key
minion-01
Rejected Keys:

[root@master salt]# salt-key -a minion-01
The following keys are going to be accepted:
Unaccepted Keys:
minion-01
Proceed? [n/Y] y    #Y确认添加
Key for minion minion-01 accepted.  #添加成功
[root@master salt]# salt-key 
Accepted Keys:
minion-01
Denied Keys:
Unaccepted Keys:
Rejected Keys:
[root@master salt]#
```

#### 联通性测试

```shell
[root@master salt]# salt 'minion-01' test.ping
minion-01:
    True   #返回结果表示成功
[root@master salt]# 
```



## 常用命令

### salt

> 该命令执行salt的执行模块,通常在master端运行.常用命令

```shell
# salt [option] '<target>' <function> [arguments]

salt 'minion-01' cmd.run 'ip addr'
```



### salt-run

> 该命令执行runner(salt自带或者自定义的，)，通常在master端执行，比如经常用到的manage

```shell
# salt-run [options] [runner.func]

salt-run manage.status   ##查看所有minion状态
salt-run manage.down     ##查看所有没在线minion
salt-run manage.up       ##查看所有在线minion
```

### salt-key

> 密钥管理，通常在master端执行

```shell
# salt-key [options]
salt-key -L              ##查看所有minion-key
salt-key -a <key-name>   ##接受某个minion-key
salt-key -d <key-name>   ##删除某个minion-key
salt-key -A              ##接受所有的minion-key
salt-key -D              ##删除所有的minion-key
```



### salt-call

> 该命令通常在minion上执行，minion自己执行可执行模块，不通过master下发job

```shell
# salt-call [options] <function> [arguments]
salt-call test.ping           ##自己执行test.ping命令
salt-call cmd.run 'ifconfig'  ##自己执行cmd.run函数
```

### salt-cp

> 分发文件到minion上,不支持目录分发.运行在master

```shell
# salt-cp [options] '<target>' SOURCE DEST

salt-cp '*' testfile.html /tmp
salt-cp 'test*' index.html /tmp/a.html
```

### cmd

> 实现对远程命令的调用执行,(默认具备root权限!谨慎使用)

```shell
#获取所欲被控主机的内存使用情况
salt '*' cmd.run 'free -m'

#在wx主机上运行test.py脚本，其中script/test.py存放在file_roots指定的目录（默认是在/srv/salt,自定义在/etc/salt/master文件中定义），
#该命令会做2个动作：首先同步test.py到minion的cache目录；起床运行该脚本
salt 'minion-01' cmd.script salt://script/test.py
```

### cron

> 实现对minion的crontab控制

```shell
#查看指定被控主机、root用户的crontab操作
salt 'minion-01' cron.raw_cron root

#为指定被控主机、root用户添加/usr/local/weekly任务zuoye
salt 'minion-01' cron.set_job root '*' '*' '*' '*' 1 /usr/local/weekly 

#删除指定被控主机、root用户crontab的/usr/local/weekly任务zuoye
salt 'minion-01' cron.rm_job root /usr/local/weekly 
```



### file

> 对minion的文件操作,包括文件读写,权限,查找校验

```shell
#校验所有被控主机/etc/fstab文件的md5值是否为xxxxxxxxxxxxx,一致则返回True值
salt '*' file.check_hash /etc/fstab md5=xxxxxxxxxxxxxxxxxxxxx

#校验所有被控主机文件的加密信息，支持md5、sha1、sha224、shs256、sha384、sha512加密算法
salt '*' file.get_sum /etc/passwd md5

#修改所有被控主机/etc/passwd文件的属组、用户权限、等价于chown root:root /etc/passwd
salt '*' file.chown /etc/passwd root root

#复制所有被控主机/path/to/src文件到本地的/path/to/dst文件
salt '*' file.copy /path/to/src /path/to/dst

#检查所有被控主机/etc目录是否存在，存在则返回True,检查文件是否存在使用file.file_exists方法
salt '*' file.directory_exists /etc

#获取所有被控主机/etc/passwd的stats信息
salt '*' file.stats /etc/passwd

#获取所有被控主机/etc/passwd的权限mode，如755，644
salt '*' file.get_mode /etc/passwd

#修改所有被控主机/etc/passwd的权限mode为0644
salt '*' file.set_mode /etc/passwd 0644

#在所有被控主机创建/opt/test目录
salt '*' file.mkdir /opt/test

#将所有被控主机/etc/httpd/httpd.conf文件的LogLevel参数的warn值修改为info
salt '*' file.sed /etc/httpd/httpd.conf 'LogLevel warn' 'LogLevel info'

#给所有被控主机的/tmp/test/test.conf文件追加内容‘maxclient 100’
salt '*' file.append /tmp/test/test.conf 'maxclient 100'

#删除所有被控主机的/tmp/foo文件
salt '*' file.remove /tmp/foo
```



### network

> 返回minion的主机信息

```shell
#在指定被控主机获取dig、ping、traceroute目录域名信息
salt 'minion-01' network.dig www.qq.com
salt 'minion-01' network.ping www.qq.com
salt 'minion-01' network.traceroute www.qq.com

#获取指定被控主机的mac地址
salt 'minion-01' network.hwaddr eth0

#检测指定被控主机是否属于10.0.0.0/16子网范围，属于则返回True
salt 'minion-01' network.in_subnet 10.0.0.0/16

#获取指定被控主机的网卡配置信息
salt 'minion-01' network.interfaces

#获取指定被控主机的IP地址配置信息
salt 'minion-01' network.ip_addrs

#获取指定被控主机的子网信息
salt 'minion-01' network.subnets
```



### pkg

> minion的程序包管理,如yum, apt-get等

```shell
#为所有被控主机安装PHP环境，根据不同系统发行版调用不同安装工具进行部署，如redhat平台的yum，等价于yum -y install php
salt '*' pkg.install php

#卸载所有被控主机的PHP环境
salt '*' pkg.remove php

#升级所有被控主机的软件包
salt '*' pkg.upgrade
```



### status

```shell
salt '*' status.version
```

#### system

> 用来日常操作计算机

```bash
system.halt        #停止正在运行的系统
system.init 3      #切换到字符界面，5是图形界面
system.poweroff
system.reboot
system.shutdown
```

### systemd(service)

```bash
  service.available sshd            #查看服务是否可用
  service.disable <service name>    #设置开机启动的服务
  service.enable <service name>
  service.disabled <service name>   #查看服务是不是开机启动
  service.enabled <service name>
  service.get_disabled              #返回所有关闭的服务
  service.get_enabled               #返回所有开启的服务
  service.get_all                   #返回所有服务
  service.reload <service name>     #重新载入指定的服务
  service.restart <service name>    #重启服务
  service.start <service name>
  service.stop <service name>
  service.status <service name>
  service.force_reload <service name>  #强制载入指定的服务
```

### salt-master

```bash
# salt-master [options]
salt-master            ##前台运行master
salt-master -d         ##后台运行master
salt-master -l debug   ##前台debug输出
```

### salt-minion

```bash
# salt-minion [options]
salt-minion            ##前台运行
salt-minion -d         ##后台运行
salt-minion -l debug   ##前台debug输出
```

### target

> target也就是目标,目的.指定master命令应该对谁执行

- 正则匹配

```bash
[root@master /]# salt -E  'mini*' test.ping
minion-02:
    True
minion-01:
    True
```

- 列表匹配

```bash
[root@master ~]# salt -L minion-01,minion-02 test.ping
minion-02:
    True
minion-01:
    True
```

- grains匹配

```bash
[root@master ~]# salt -G 'os:CentOs' test.ping
minion-02:
    True
minion-01:
    True
```

- 组匹配

```bash
#开启master 的default_include
vim /etc/salt/master.d/nodegroup.conf 
#写到master中也是这个格式
nodegroups:
 test1: 'L@test1,test2 or test3*'
 test2: 'G@os:CenOS or test2'

salt -N test1 test.ping   #-N指定groupname

在top file中使用nodegroups

'test1':
 - match: nodegroup     ##没s,匹配的是文件
 - webserver
```

```bash
[root@master ~]# salt -N nodegroups test.ping
minion-02:
    True
minion-01:
    True
#组需要在master中预先定义
```

- 复合匹配  `salt -C 'G@os:MacOS or L@Minion1'`
- Pillar匹配 `salt -I 'key:value' test.ping`
- CIDR匹配 `salt -S '192.168.1.0/24' test.ping`



## pillar

> Pillar在salt中是非常重要的组成部分，利用它可以完成很强大的功能，它可以指定一些信息到指定的minion上，不像grains一样是分发到所有Minion上的，它保存的数据可以是动态的,Pillar以sls来写的，格式是键值
>
> 适用
>
> 1.比较敏感的数据，比如密码，key等
>
> 2.特殊数据到特定Minion上
>
> 3.动态的内容
>
> 4.其他数据类型

**查看所有**

```bash
salt '*' pillar.items
```

**查看某个**

```bash
salt '*' pillar.item KEY
#可以取到更小粒度的
salt '*' pillar.get <key>:<key> 
```

#### 编写pillar

> 指定pillar_roots
>
> 默认是/srv/pillar/(可通过修改master配置文件修改),建立目录

**top.sls**

```bash
base:           #指定环境
  '*':          #target
    - test1     #引用test1.sls 或者test1/init.sls
    
#通过分组名匹配，
base:
  group1:
    - match: nodegroup    #必须要有 - match: nodegroup  
    - webserver  

#通过grain模块匹配的示例
base:
  'os:CentOS':
    - match: grain   #必须要有- match: grain
    - webserver
    
```

**test1.sls**

```bash
name: test1
user: lzl
```

**刷新**  pillar数据

```bash
salt '*' saltutil.refresh_pillar
```

**查看结果**

```bash
[root/srv/pillar] ]$salt 'minion-01' pillar.items
minion-01:
    ----------
    name:
        test1
    user:
        lzl
[root/srv/pillar] ]$
```



> [saltstack学习](https://www.jianshu.com/p/624b9cf51c64)
>
> 
