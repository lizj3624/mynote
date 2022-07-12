---
title: "Linux下C/C++程序崩溃错误定位思路"
date: 2022-07-12T12:27:22+08:00
tags:
   - Linux
   - addr2line 
   - crash
categories:
   - Linux
   - addr2line 
   - crash
toc: true
---

Linux下开发的C/C++程序崩溃后错误定位的思路。

### coredump定位
Linux下开发的C/C++崩溃后可以产生coredump文件，通过coredump文件可以定位崩溃点。但是必须前期设置coredump，
详细的设置步骤和定位方法可以参考另一篇文章: [Linux下的cordump设置和使用](https://lizj3624.github.io/post/cordump/)。

### addr2line定位
有时候上线环境没有开启cordump，又着急定位问题，可以通过`addr2line`粗略定位崩溃点。

下面我们详细说明这个思路：

一般Linux下程序崩溃时会有`segfault`信息，可以通过`dmesg`命令查看，以nginx崩溃为例说一下
```shell
nginx[55279]: segfault at 48 ip 000000000049259c sp 00007fffec5b7350 error 4 in nginx[400000+1ed000]
nginx[55276]: segfault at 48 ip 000000000049259c sp 00007fffec5b7350 error 4 in nginx[400000+1ed000]
```
简单说一下`segfault`信息:
1. ip 000000000049259c 发生错误时指令的地址

2. sp 00007fffec5b7350 堆栈指针 

3. error 4 为用户态内存读操作访问出界，读非法地址，可以对照参考：
> bit2:值为1表示是用户态程序内存访问越界，值为0表示是内核态程序内存访问越界   
> bit1: 值为1表示是写操作导致内存访问越界，值为0表示是读操作导致内存访问越界    
> bit0: 值为1表示没有足够的权限访问非法地址的内容，值为0表示访问的非法地址根本没有对应的页面，也就是无效地址

Linux下addr2line命令用于将程序指令地址转换为所对应的函数名、以及函数所在的源文件名和行号。
当含有调试信息(-g)的执行程序出现crash时(core dumped)，可使用addr2line命令快速定位出错的位置。

定位命令查看, 需要依赖debuginfo包
```shell
addr2line -e /usr/lib/debug/nginx/sbin/nginx.debug -fCi 000000000049259c
ngx_http_v2_process_request_body
/usr/src/debug/nginx/build/nginx-1.15.8/src/http/v2/ngx_http_v2.c:3975
```

### 引用
1. [DMESG和ADDR2LINE定位SEGFAULT](https://www.cnblogs.com/qhbk/p/7666324.html)

2. [拒绝超大coredump-用backtrace和addr2line搞定异常函数栈](https://zhuanlan.zhihu.com/p/31630417)
