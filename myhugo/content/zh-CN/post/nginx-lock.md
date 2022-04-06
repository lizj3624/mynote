---
title: "Nginx的锁机制"
date: 2022-04-06T19:23:06+08:00
tags:
   - nginx
   - 源码 
categories:
   - nginx
   - 源码 
toc: true
---
> 我们项目中nginx的dyups模块在大量更新时，有性能瓶颈，经排查发现时用的自旋锁导致性能瓶颈，dyups使用ngx_shmtx_lock和ngx_shmtx_trylock两种锁，
> 用户通过参数可以设置选择修改，发现ngx_shmtx_lock性能较差，其中ngx_shmtx_trylock这个锁每次执行的时候只是去抢一下锁，当没有抢到则立即返回失败。
> ngx_shmtx_lock锁相当于一个自旋锁（splinlock）当进程执行到此处的时候会是一个死循环的去进行抢锁，当然中间加了一些自己的限制机制， 
> 当抢一次失败的时候会增加下一次去抢所的时间间隔，而且抢了很多次后还是抢不到锁的时候就会先让出cpu一会
> 此处调用ngx_sched_yield让其他的程序先执行－－－找一个优先级等于或是高于当前程序的去执行，相比之下ngx_shmtx_trylock的性能高多了。
> 这两个锁都是使用了cpu级别的原子操作，不会发生上下文的切换。
> 在此研究一下nginx的锁机制，这是从CSDN摘录的文件。

nginx使用共享内存来进行进程间通信，那么就需要一把锁来确保进程通信的正确，在nginx中通过判断操作系统是否支持相应的锁来进行选定锁的类型，分为:
1. 原子锁

2. 信号量互斥锁

3. 文件锁(互斥锁)

4. 自旋锁(多处理器时使用)

ngx_shmtx.h
```c
typedef struct {
    ngx_atomic_t   lock;
#if (NGX_HAVE_POSIX_SEM)	//定义使用信号量
    ngx_atomic_t   wait;
#endif
} ngx_shmtx_sh_t;

typedef struct {
#if (NGX_HAVE_ATOMIC_OPS)		//原子锁
    ngx_atomic_t  *lock;
#if (NGX_HAVE_POSIX_SEM)		//信号量互斥锁
    ngx_atomic_t  *wait;
    ngx_uint_t     semaphore;
    sem_t          sem;
#endif
#else							//文件锁
    ngx_fd_t       fd;
    u_char        *name;
#endif
    ngx_uint_t     spin;		//多处理器时才会使用,即ngx_ncpu>1
} ngx_shmtx_t;
```

ngx_shmtx.c
```c
ngx_int_t
ngx_shmtx_create(
ngx_shmtx_t *mtx, ngx_shmtx_sh_t *addr, u_char *name)
{
    mtx->lock = &addr->lock;	//获取位于共享内存中的lock区

    if (mtx->spin == (ngx_uint_t) -1) {	//判断是否支持信号量互斥锁
        return NGX_OK;
    }

    mtx->spin = 2048;

#if (NGX_HAVE_POSIX_SEM)

    mtx->wait = &addr->wait;

    /**
    *信号量相关的头文件
    *#include <semaphore.h>
    *int sem_init(sem_t *sem,int pshared,unsigned int value);
    *当pshared为0代表该信号量用于多线程间的同步
    *大于0表示用于多个相关进程间的同步
    *value为初始化sem的值
    **/

    if (sem_init(&mtx->sem, 1, 0) == -1) {	
        ngx_log_error(NGX_LOG_ALERT, ngx_cycle->log, 
        			  ngx_errno, "sem_init() failed");
    } else {
        mtx->semaphore = 1;
    }

#endif

    return NGX_OK;
}

void
ngx_shmtx_destroy(ngx_shmtx_t *mtx)
{
#if (NGX_HAVE_POSIX_SEM)
	//当支持信号量时才能生效
    if (mtx->semaphore) {	//mtx->semaphore初始化成功时为1
        if (sem_destroy(&mtx->sem) == -1) {	//清理信号量
            ngx_log_error(NGX_LOG_ALERT, ngx_cycle->log, 
            	ngx_errno, "sem_destroy() failed");
        }
    }

#endif
}

void
ngx_shmtx_lock(ngx_shmtx_t *mtx)
{
    ...
    for ( ;; ) {
		//是否持有锁
        if (*mtx->lock == 0 && 
        	ngx_atomic_cmp_set(mtx->lock, 0, ngx_pid)) {
            return;
        }
		//有多个处理器时启用自旋锁
        if (ngx_ncpu > 1) {

            for (n = 1; n < mtx->spin; n <<= 1) {

                for (i = 0; i < n; i++) {
                	/*ngx_cpu_pause是汇编代码
                	__asm__ ("pause")的拓展*/
                    ngx_cpu_pause();
                }

                if (*mtx->lock == 0
                    && ngx_atomic_cmp_set(mtx->lock, 0, ngx_pid))
                {
                    return;
                }
            }
        }
#if (NGX_HAVE_POSIX_SEM)

        if (mtx->semaphore) {
        	//具有原子操作的__sync_fetch_and_add(mtx->wait, 1)
            (void) ngx_atomic_fetch_add(mtx->wait, 1);

            if (*mtx->lock == 0 
            && ngx_atomic_cmp_set(mtx->lock, 0, ngx_pid)) {
            	//获得锁后共享内存wait减一
                (void) ngx_atomic_fetch_add(mtx->wait, -1);
                return;
            }
            ...

            while (sem_wait(&mtx->sem) == -1) {
                ngx_err_t  err;

                err = ngx_errno;
				/**若err=NGX_EINTR
				*此时的信号量等待被打断而不是出错
				*向log写入信息后继续等待
				*/
                if (err != NGX_EINTR) {
					...
					}
            }
            ...
            continue;
        }

#endif
		//程序挂起一段时间，让出处理器，当使用信号量时无效
        ngx_sched_yield();	
    }
}
```
1. 如果没有使用信号量就和之前的自旋锁没有什么区别。
2. 如果使用了信号量，并且自旋锁一轮过来(spin消耗完全)都没有加锁成功，则会尝试最后一次加锁，如果加锁失败则使用信号量进行休眠等待唤醒。

```c
void
ngx_shmtx_unlock(ngx_shmtx_t *mtx)
{
    if (mtx->spin != (ngx_uint_t) -1) {
    //判断该进程是否能够进入休眠状态
	...
    }
	//将lock置为0，释放了锁
    if (ngx_atomic_cmp_set(mtx->lock, ngx_pid, 0)) {
        ngx_shmtx_wakeup(mtx);
    }
}


ngx_uint_t
ngx_shmtx_force_unlock(ngx_shmtx_t *mtx, ngx_pid_t pid)
{
    //强迫进程释放锁
    ...
    if (ngx_atomic_cmp_set(mtx->lock, pid, 0)) {
        ngx_shmtx_wakeup(mtx);
        return 1;
    }

    return 0;
}


static void
ngx_shmtx_wakeup(ngx_shmtx_t *mtx)
{
#if (NGX_HAVE_POSIX_SEM)
    ngx_atomic_uint_t  wait;

    if (!mtx->semaphore) {
    	//判断是否使用了信号量
        return;
    }

    for ( ;; ) {

        wait = *mtx->wait;
		//无进程在等待直接返回
        if ((ngx_atomic_int_t) wait <= 0) {
            return;
        }
		//通过原子操作减少当前等待的进程数
        if (ngx_atomic_cmp_set(mtx->wait, wait, wait - 1)) {
            break;
        }
    }
    ...

    if (sem_post(&mtx->sem) == -1) {
    	//释放了信号量锁
		...
    }

#endif
}
```

到此可以看出nginx如何实现高效的原子互斥锁， 即通过信号量使得进程去获得一把锁，当一个进程使用sem_post释放了锁，
另一个进程sem_wait得到了锁，若是处于sem当前值等于0时，则进程进入睡眠状态等待唤醒。

nginx实现的另一种锁，文件锁，即不支持原子锁时，通过文件锁保证进程通信的准确。
```c
ngx_int_t
ngx_shmtx_create(ngx_shmtx_t *mtx,
ngx_shmtx_sh_t *addr, u_char *name)
{
    if (mtx->name) {
		//判断mtx->name已经初始化
        if (ngx_strcmp(name, mtx->name) == 0) {	
            mtx->name = name;
            return NGX_OK;
        }
		//如果已经存在打开的文件则关闭
        ngx_shmtx_destroy(mtx);
    }
	//打开文件名为name的文件
    mtx->fd = ngx_open_file(name, NGX_FILE_RDWR, 
    						NGX_FILE_CREATE_OR_OPEN,
                            NGX_FILE_DEFAULT_ACCESS);

    if (mtx->fd == NGX_INVALID_FILE) {
		...
    }

    if (ngx_delete_file(name) == NGX_FILE_ERROR) {
		/*ngx_delete_file为ulink
		 *由于并不需要真实文件，只需要文件fd在内核中存在即可
		 */
		...
    }

    mtx->name = name;

    return NGX_OK;
}

void
ngx_shmtx_destroy(ngx_shmtx_t *mtx)
{
    if (ngx_close_file(mtx->fd) == NGX_FILE_ERROR) {
		//ngx_close_file()即为close()
		...
    }
}

ngx_uint_t
ngx_shmtx_trylock(ngx_shmtx_t *mtx)
{
    ngx_err_t  err;
	//文件上锁，此时文件锁并不会使阻塞进程
    err = ngx_trylock_fd(mtx->fd);
    ...

#if __osf__ /* Tru64 UNIX */

    if (err == NGX_EACCES) {
        return 0;
    }

#endif
    ...
    return 0;
}

void
ngx_shmtx_lock(ngx_shmtx_t *mtx)
{
    ngx_err_t  err;

    err = ngx_lock_fd(mtx->fd);

    if (err == 0) {
        return;
    }
    ...
}
```

`ngx_shmtx_lock`与`ngx_shmtx_trylock`区别为`ngx_shmtx_lock`在上锁时，
即`fcntl(fd, F_SETLKW, &fl)`中`cmd`的字段为`F_SETLKW`，将会在不能获取锁时阻塞进程，而`ngx_shmtx_trylock`中的字段为`F_SETLK`并不会阻塞进程。

```c
void
ngx_shmtx_unlock(ngx_shmtx_t *mtx)
{
    ...
	//释放锁将fcntl中cmd字段置为UNLCK即可
    err = ngx_unlock_fd(mtx->fd);

    if (err == 0) {
        return;
    }
    ...
}
//强制释放锁
ngx_uint_t
ngx_shmtx_force_unlock(ngx_shmtx_t *mtx, ngx_pid_t pid)
{
    return 0;
}
```
nginx实现的锁有个共同点即是在其他进程始终不放弃锁时，一段时间后将会强制获得该锁。
有关信号量的内容可以通过搜索Linux信号量。

引用
1. [nginx之nginx_shmtx(锁机制)](https://blog.csdn.net/weixin_43352959/article/details/107500412)

2. [nginx锁(spinlock,ngx_shmtx_t)](https://blog.csdn.net/qq_41252394/article/details/111472720)
