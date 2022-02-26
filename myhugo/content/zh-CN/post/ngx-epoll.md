---
title: "nginx epoll惊群"
date: 2022-02-26T08:23:43+08:00
tags:
   - nginx 
   - epoll 
categories:
   - nginx 
   - epoll
toc: true
---

> nginx是目前比较流行的高性能的负载均衡，反向代理，静态web服务器，它的高性能主要是基于epoll(Linux)的事件框架。   
> nginx是开源，我们可以通过阅读源码分析它的epoll事件的实现。    
> 还有epoll存在惊群的问题，看看nginx是如何解决的这个问题的。    
> 从网络上收集整理一些资料，在此汇总一下，以便查阅。

## 惊群现象

首先，我们看看维基百科对[惊群的定义](https://en.wikipedia.org/wiki/Thundering_herd_problem),简而言之，
惊群现象（thundering herd）就是当多个进程和线程在同时阻塞等待同一个事件时，如果这个事件发生，会唤醒所有的进程，
但最终只可能有一个进程/线程对该事件进行处理，其他进程/线程会在失败后重新休眠，这种性能浪费就是惊群。

## accept惊群

考虑如下场景：

主进程创建`socket、bind、 listeni`之后，`fork`出多个子进程，每个子进程都开始循环处理`accept`这个`socket`。
每个进程都阻塞在`accpet`上，当一个新的连接到来时，所有的进程都会被唤醒，但其中只有一个进程会`accept`成功，
其余皆失败，重新休眠。这就是`accept`惊群。

那么这个问题真的存在吗？

事实上，历史上，Linux的`accpet`确实存在惊群问题，但现在的内核都解决该问题了。即当多个进程/线程都阻塞在对同一个`socket`的`accept`调用上时，
当有一个新的连接到来，内核只会唤醒一个进程，其他进程保持休眠，压根就不会被唤醒。

测试验证代码:
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <sys/wait.h>
#include <stdio.h>
#include <string.h>
#define PROCESS_NUM 10
int main()
{
    int fd = socket(PF_INET, SOCK_STREAM, 0);
    int connfd;
    int pid;
    char sendbuff[1024];
    struct sockaddr_in serveraddr;
    serveraddr.sin_family = AF_INET;
    serveraddr.sin_addr.s_addr = htonl(INADDR_ANY);
    serveraddr.sin_port = htons(1234);
    bind(fd, (struct sockaddr*)&serveraddr, sizeof(serveraddr));
    listen(fd, 1024);
    int i;
    for(i = 0; i < PROCESS_NUM; i++)
    {
        int pid = fork();
        if(pid == 0)
        {
            while(1)
            {
                connfd = accept(fd, (struct sockaddr*)NULL, NULL);
                snprintf(sendbuff, sizeof(sendbuff), "accept PID is %d\n", getpid());

                send(connfd, sendbuff, strlen(sendbuff) + 1, 0);
                printf("process %d accept success!\n", getpid());
                close(connfd);
            }
        }
    }
    int status;
    wait(&status);
    return 0;
}
```
当我们对该服务器发起连接请求（用 telnet/curl 等模拟）时，会看到只有一个进程被唤醒。

关于 accept 惊群的一些帖子或文章：

1. [Does the Thundering Herd Problem exist on Linux anymore?](https://stackoverflow.com/questions/2213779/does-the-thundering-herd-problem-exist-on-linux-anymore)
2. [历史上解决 linux accept 惊群的补丁讨论](http://www.citi.umich.edu/projects/linux-scalability/reports/accept.html)

> 其实，在linux2.6内核上，accept系统调用已经不存在惊群了（至少我在2.6.18内核版本上已经不存在）

## epoll惊群

如上所述，`accept`已经不存在惊群问题，但`epoll`上还是存在惊群问题。即如果多个进程/线程阻塞在监听同一个`listening socket fd`的`epoll_wait`上，
当有一个新的连接到来时，所有的进程都会被唤醒。

考虑如下场景：

主进程创建`socket、bind、 listen`后，将该`socket`加入到`epoll`中，然后 `fork`出多个子进程，每个进程都阻塞在`epoll_wait`上，如果有事件到来，
则判断该事件是否是该`socket`上的事件，如果是说明有新的连接到来了，则进行`accept`操作。为了简化处理，
忽略后续的读写以及对`accept`返回的新的套接字的处理，直接断开连接。

那么，当新的连接到来时，是否每个阻塞在`epoll_wait`上的进程都会被唤醒呢？

很多博客中提到，测试表明虽然`epoll_wait` 不会像`accept`那样只唤醒一个进程/线程，但也不会把所有的进程/线程都唤醒。
例如这篇文章：[关于多进程 epoll 与 “惊群”问题](https://www.jianshu.com/p/362b56b573f4)。

测试验证代码:
```c
#include <sys/types.h>
#include <sys/socket.h>
#include <sys/epoll.h>
#include <netdb.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>
#include <errno.h>
#include <sys/wait.h>
#define PROCESS_NUM 10
static int
create_and_bind (char *port)
{
    int fd = socket(PF_INET, SOCK_STREAM, 0);
    struct sockaddr_in serveraddr;
    serveraddr.sin_family = AF_INET;
    serveraddr.sin_addr.s_addr = htonl(INADDR_ANY);
    serveraddr.sin_port = htons(atoi(port));
    bind(fd, (struct sockaddr*)&serveraddr, sizeof(serveraddr));
    return fd;
}

static int
make_socket_non_blocking (int sfd)
{
    int flags, s;

    flags = fcntl (sfd, F_GETFL, 0);
    if (flags == -1)
    {
        perror ("fcntl");
        return -1;
    }

    flags |= O_NONBLOCK;
    s = fcntl (sfd, F_SETFL, flags);
    if (s == -1)
    {
        perror ("fcntl");
        return -1;
    }

    return 0;
}

#define MAXEVENTS 64

int
main (int argc, char *argv[])
{
    int sfd, s;
    int efd;
    struct epoll_event event;
    struct epoll_event *events;

    sfd = create_and_bind("1234");
    if (sfd == -1)
        abort ();

    s = make_socket_non_blocking (sfd);
    if (s == -1)
        abort ();

    s = listen(sfd, SOMAXCONN);
    if (s == -1)
    {
        perror ("listen");
        abort ();
    }

    efd = epoll_create(MAXEVENTS);
    if (efd == -1)
    {
        perror("epoll_create");
        abort();
    }

    event.data.fd = sfd;
    //event.events = EPOLLIN | EPOLLET;
    event.events = EPOLLIN;
    s = epoll_ctl(efd, EPOLL_CTL_ADD, sfd, &event);
    if (s == -1)
    {
        perror("epoll_ctl");
        abort();
    }

    /* Buffer where events are returned */
    events = calloc(MAXEVENTS, sizeof event);
	        int k;
    for(k = 0; k < PROCESS_NUM; k++)
    {
        int pid = fork();
        if(pid == 0)
        {

            /* The event loop */
            while (1)
            {
                int n, i;
                n = epoll_wait(efd, events, MAXEVENTS, -1);
                printf("process %d return from epoll_wait!\n", getpid());
	                                   /* sleep here is very important!*/
                //sleep(2);
	                                   for (i = 0; i < n; i++)
                {
                    if ((events[i].events & EPOLLERR) || (events[i].events & EPOLLHUP) || (!(events[i].events &                                    EPOLLIN)))
                    {
                        /* An error has occured on this fd, or the socket is not
                        ready for reading (why were we notified then?) */
                        fprintf (stderr, "epoll error\n");
                        close (events[i].data.fd);
                        continue;
                    }
                    else if (sfd == events[i].data.fd)
                    {
                        /* We have a notification on the listening socket, which
                        means one or more incoming connections. */
                        struct sockaddr in_addr;
                        socklen_t in_len;
                        int infd;
                        char hbuf[NI_MAXHOST], sbuf[NI_MAXSERV];

                        in_len = sizeof in_addr;
                        infd = accept(sfd, &in_addr, &in_len);
                        if (infd == -1)
                        {
                            printf("process %d accept failed!\n", getpid());
                            break;
                        }
                        printf("process %d accept successed!\n", getpid());

                        /* Make the incoming socket non-blocking and add it to the
                        list of fds to monitor. */
                        close(infd);
                    }
                }
            }
        }
    }
    int status;
    wait(&status);
    free (events);
    close (sfd);
    return EXIT_SUCCESS;
}
```
发现确实如上面那篇博客里所说，当我模拟发起一个请求时，只有一个或少数几个进程被唤醒了。

也就是说，到目前为止，还没有得到一个确定的答案。但后来，在下面这篇博客中看到这样一个[评论](http://blog.csdn.net/spch2008/article/details/18301357)

> 这个总结，需要进一步阐述，看上去是只有4个进程唤醒了，而事实上，其余进程没有被唤醒的原因是你的某个进程已经处理完这个accept，内核队列上已经没有这个事件，
> 无需唤醒其他进程。你可以在epoll获知这个accept事件的时候，不要立即去处理，而是sleep下，这样所有的进程都会被唤起。

看到这个评论后，我顿时如醍醐灌顶，重新修改了上面的测试程序，即在`epoll_wait`返回后，加了个`sleep`语句，这时再测试，果然发现所有的进程都被唤醒了。

所以，`epoll_wait`上的惊群确实是存在的。

## 为什么Kernel不处理Epoll惊群

看到这里，我们可能有疑惑了，为什么内核对`accept`的惊群做了处理，而现在仍然存在`epoll`的惊群现象呢？

我想，应该是这样的：

`accept`确实应该只能被一个进程调用成功，内核很清楚这一点。但`epoll`不一样，他监听的文件描述符，除了可能后续被`accept`调用外，还有可能是其他网络`IO`事件的，
而其他`IO`事件是否只能由一个进程处理，是不一定的，内核不能保证这一点，这是一个由用户决定的事情，例如可能一个文件会由多个进程来读写。所以对`epoll`的惊群，内核则不予处理。

## nginx解决惊群的方法

### nginx的epoll框架

1. nginx主进程解析配置，将listen指令初始化到全局变量`ngx_cycle`的`listening`数组之中。此时监听套接字的创建、绑定工作早已完成。

2. nginx主进程`fork`出多个子进程(worker进程), 每个子进程执行`ngx_worker_process_init`, 为每个子进程创建`epoll`句柄。

3. 每个子进程执行`ngx_process_events_and_timers`，这就进入到事件处理的核心逻辑了，如果开启` accept_mutex`，每个进程争抢锁, `epoll_wait`等待处理网络事件。

### accept_mutex锁

如果开启了`accept_mutex`锁，每个`worker`都会先去抢自旋锁，只有抢占成功了，才把`socket`加入到`epoll`中，`accept`请求后释放锁, `accept_mutex`锁也有负载均衡的作用。
`accept_mutex`效率低下，特别是在长连接的时候。因为长连接时，一个进程长时间占用`accept_mutex`锁，使得其它进程得不到`accept`的机会。因此不建议使用，默认是关闭的。

### EPOLLEXCLUSIVE标识

EPOLLEXCLUSIVE是`4.5+`内核新添加的一个`epoll`的标识，Ngnix 在`1.11.3`之后添加了`NGX_EXCLUSIVE_EVENT`。
`EPOLLEXCLUSIVE`标识会保证一个事件发生时候只有一个线程会被唤醒，以避免多侦听下的“惊群”问题。
不过任一时候只能有一个工作线程调用`accept`，限制了真正并行的吞吐量。

### SO_REUSEPORT 选项

`SO_REUSEPORT`是惊群最好的解决方法，Ngnix在`1.9.1`中加入了这个选项，每个`worker`都有自己的`socket`，这些`socket`都`bind`同一个端口。
当新请求到来时，内核根据四元组信息进行负载均衡，非常高效。

> Linux 3.9版本的内核对reuseport做了支持，在4.6版本内核做了优化，详细参看[关于Linux UDP/TCP reuseport 二三事](https://blog.csdn.net/dog250/article/details/80458669)

## 总结

现在我们对惊群及`Nginx`的处理总结如下：

1. `accept`不会有惊群，`epoll_wait`才会。

2. `Nginx`的`accept_mutex`，并不是解决`accept`惊群问题，而是解决`epoll_wait`惊群问题。

3. 说`Nginx`解决了`epoll_wait`惊群问题，也是不对的，它只是控制是否将监听套接字加入到`epoll`中。
   监听套接字只在一个子进程的`epoll`中，当新的连接来到时，其他子进程当然不会惊醒了。

## 引用文章

1. [accept与epoll惊群](https://pureage.info/2015/12/22/thundering-herd.html)

2. [Nginx是如何解决epoll惊群的](https://ld246.com/article/1588731832846)

3. [“惊群”，看看nginx是怎么解决它的](https://blog.csdn.net/russell_tao/article/details/7204260)

4. [关于Linux UDP/TCP reuseport 二三事](https://blog.csdn.net/dog250/article/details/80458669)

5. [重新实现reuseport逻辑，实现一致性哈希](https://blog.csdn.net/dog250/article/details/89268404)
