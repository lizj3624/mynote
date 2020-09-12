## OpenResty源码剖析-lua代码的加载

### **OpenResty是什么**

OpenResty是一个基于 Nginx与Lua的高性能 Web 平台,通过把lua嵌入到Nginx中，使得我们可以用轻巧的lua语言进行nginx的相关开发，处理高并发，扩展性极高的动态 Web 应用。

大家知道lua_code_cache 开关用于控制是否缓存*_by_lua_file对应的文件里的lua代码

lua_code_cache off的情况下，跟请求有关的阶段，在每次有请求来的时候，都会重新加载最新的lua文件，这样我们修改完代码之后就不用通过reload来更新代码了

而*_by_lua_block、*_by_lua和init_by_lua_file里的代码(init_by_lua阶段和具体请求无关)，如果修改的内容涉及这几个，仍需要通过reload来更新代码

那openresty是如何实现这些，如何完成加载代码，和代码缓存的呢？

### **Nginx配置**

假设Nginx相关的配置如下所示

```conf
lua_code_cache off;
location ~ ^/api/([-_a-zA-Z0-9/]+) { 
    content_by_lua_file lua/$1.lua;
}
```

当来到的请求符合` ^/api/([-_a-zA-Z0-9/]`时，会在`NGX_HTTP_CONTENT_PHASE` `HTTP`请求内容阶段交给 `lua/$1.lua`来处理

比如:

```shell
/api/addition           交给 lua/addition.lua 处理

/api/substraction    交给 lua/substraction .lua 处理
```

### **请求的处理**

`content_by_lua_file`对应的请求来临时，执行流程为`ngx_http_lua_content_handler -> ngx_http_lua_content_handler_file-> ngx_http_lua_content_by_chunk`

#### **配置项相关** 

```c
{ 
    ngx_string("content_by_lua_file"),
    NGX_HTTP_LOC_CONF|NGX_HTTP_LIF_CONF|NGX_CONF_TAKE1,
    ngx_http_lua_content_by_lua,
    NGX_HTTP_LOC_CONF_OFFSET,
    0,
    (void *) ngx_http_lua_content_handler_file 
}

 char *
 ngx_http_lua_content_by_lua(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
 {
 ...
     llcf->content_handler = (ngx_http_handler_pt) cmd->post;//设置回调函数为ngx_http_lua_content_handler_file
 ...
     clcf->handler = ngx_http_lua_content_handler;//使用按需挂载处理函数的方式挂载处理函数，处理函数为ngx_http_lua_content_handler

     return NGX_CONF_OK;
 }
```

#### **处理函数**

```c
ngx_int_t
ngx_http_lua_content_handler(ngx_http_request_t *r)137 {
...
    ctx = ngx_http_get_module_ctx(r, ngx_http_lua_module); //获取请求在ngx_http_module模块对应的上下文结构

    dd("ctx = %p", ctx);

    if (ctx == NULL) {
        ctx = ngx_http_lua_create_ctx(r); //如果之前没有设置过上下文，调用ngx_http_lua_create_ctx创建上下文结构
        if (ctx == NULL) {
            return NGX_HTTP_INTERNAL_SERVER_ERROR;
        }
    }

    dd("entered? %d", (int) ctx->entered_content_phase);
...
    return llcf->content_handler(r); //调用ngx_http_content_handler_file函数
}
```

#### **创建上下文结构**

```c
ngx_http_lua_create_ctx(ngx_http_request_t *r)
{
...
    if (!llcf->enable_code_cache && r->connection->fd != (ngx_socket_t) -1) { //如果lua_code_cache off
...
        L = ngx_http_lua_init_vm(lmcf->lua, lmcf->cycle, r->pool, lmcf,
                                 r->connection->log, &cln); //为请求初始化一个新的lua_state
        ctx->vm_state = cln->data;

    } else {
        ctx->vm_state = NULL; //不分配新的lua_state,这样所有请求都会使用ngx_http_lua_module模块的lua_state
    }
}
```

276行 如果关闭了lua代码缓存，那么openresty就会为每一个请求创建一个新的`lua_state`并设置相关的字段，最后赋值给`ctx->vm_state`,

ctx->vm_state的类型如下 

```c
typedef struct {
     lua_State       *vm;
     ngx_int_t        count;
 } ngx_http_lua_vm_state_t;
```

#### **回调函数**

```c
ngx_int_t
ngx_http_lua_content_handler_file(ngx_http_request_t *r)
{
...
    script_path = ngx_http_lua_rebase_path(r->pool, eval_src.data,
                                                    eval_src.len); //获取lua文件的路径
    L = ngx_http_lua_get_lua_vm(r, NULL); //获得lua_state,如果请求有自己的lua_state则使用请求自己的lua_state，否则使用ngx_http_lua_module模块的lua_state

   /*  load Lua script file (w/ cache)        sp = 1 */
   rc = ngx_http_lua_cache_loadfile(r->connection->log, L, script_path,
                                    llcf->content_src_key); //加载代码
...
    return ngx_http_lua_content_by_chunk(L, r); //创建协程执行代码的函数
} 
```

#### **代码加载**

```c
ngx_int_t
ngx_http_lua_cache_loadfile(ngx_log_t *log, lua_State *L,
    const u_char *script, const u_char *cache_key)
{
...
    rc = ngx_http_lua_cache_load_code(log, L, (char *) cache_key);
    if (rc == NGX_OK) {//代码在全局变量table中存在，则返回
...
        return NGX_OK;
    }
...
    rc = ngx_http_lua_clfactory_loadfile(L, (char *) script);//
...
    rc = ngx_http_lua_cache_store_code(L, (char *) cache_key);
...
    return NGX_OK;

error:
...
} 
```

代码加载分成3步完成

`ngx_http_lua_cache_load_code`从`lua_state`的全局变量`table`中加载代码，如果全局缓存中有就返回

`ngx_http_lua_clfactory_loadfile`用自定义的函数从文件中加载代码

`ngx_http_lua_cache_store_code` 把代码存放到`lua_state`的全局变量`table`中

#### **尝试从全局变量table中加载代码**

```c
static ngx_int_t
ngx_http_lua_cache_load_code(ngx_log_t *log, lua_State *L,
    const char *key)
{
...
    /*  get code cache table */
    lua_pushlightuserdata(L, &ngx_http_lua_code_cache_key);
    lua_rawget(L, LUA_REGISTRYINDEX);    /*  sp++ */
...
    lua_getfield(L, -1, key);    /*  sp++ */

    if (lua_isfunction(L, -1)) {
        /*  call closure factory to gen new closure */
        rc = lua_pcall(L, 0, 1, 0);
        if (rc == 0) {
...
            return NGX_OK;
        }
...
        return NGX_ERROR;
    }
...
    return NGX_DECLINED;
}
```

42-52行，相当于LUA_REGISTRYINDEX`[‘ngx_http_lua_code_cache_key’][‘key’]`以`ngx_http_lua_code_cache_key`为索引从全局注册表表中查找key对于的value

54-61行，如果value存在并且为一个函数，因为这里的函数体是`return function() … end`包裹的  所以在56行需要再调用`lua_pcall`执行下，以获得返回的函数并将返回的函数结果放到栈顶，最终将`LUA_REGISTRYINDEX`从栈中移除

如果代码缓存关闭的时候，openresty会为每一个请求创建一个新的`lua_state`，这样请求来临的时候在全局变量table中找不到对应的代码缓存，需要到下一步`ngx_http_lua_clfactory_loadfile`中读取文件加载代码

如果代码缓存打开的时候，openresty会使用`ngx_http_lua_module`全局的`lua_state`，这样只有新的lua文件，在首次加载时需要到`ngx_http_lua_clfactory_loadfile`中读取文件加载代码，第二次来的时候便可以在`lua_state`对应的全局变量table中找到了

#### **从文件中读取代码**

```c
ngx_int_t
ngx_http_lua_clfactory_loadfile(lua_State *L, const char *filename)
{
...
    lf.begin_code.ptr = CLFACTORY_BEGIN_CODE;
    lf.begin_code_len = CLFACTORY_BEGIN_SIZE;
    lf.end_code.ptr = CLFACTORY_END_CODE;
    lf.end_code_len = CLFACTORY_END_SIZE;
...
    lf.f = fopen(filename, "r");
...
    status = lua_load(L, ngx_http_lua_clfactory_getF, &lf,
                      lua_tostring(L, -1));
...
    return status;
}
```

`#define CLFACTORY_BEGIN_CODE "return function() "`

`#define CLFACTORY_END_CODE "\nend"`

700行用自定义的`ngx_http_lua_clfactory_get`函数读取lua代码

并在原有代码的开头加上了`return function()`结束处加上了`\nend`

#### **缓存代码**

```c
ngx_http_lua_cache_store_code(lua_State *L, const char *key)
{
...
    lua_pushlightuserdata(L, &ngx_http_lua_code_cache_key);
    lua_rawget(L, LUA_REGISTRYINDEX);
...
    lua_pushvalue(L, -2); /* closure cache closure */
    lua_setfield(L, -2, key); /* closure cache */
...
    lua_pop(L, 1); /* closure */
...
    rc = lua_pcall(L, 0, 1, 0);
...
    return NGX_OK;
}
```

108-119行，相当于`LUA_REGISTRYINDEX[‘ngx_http_lua_code_cache_key’][‘key’] = function xxx`,将代码放入全局`table`中

122行，将`LUA_REGISTRYINDEX`从栈中弹出

125行，因为代码块是`return function() … end`包裹的，所以在56行需要再调用`lua_pcall`执行以获得返回的函数

### **总结**

1、当`lua_code_cache off`的情况下，openresty关闭`lua`代码缓存，为每一个请求都创建一个新的lua_state，这样每一个请求来临的时候在新创建的`lua_state`中，都在全局table的代码缓存中找不到代码，需要重新读取文件加载代码，因此可以立即动态加载新的lua脚本，而不需要`reload nginx`，但因为每个请求都需要分配新的`lua_state`,和读取文件加载代码，所以性能较差。

2、当`lua_code_cache on`的情况下，openresty打开lua代码缓存，每一个请求使用`ngx_http_lua_module`全局的`lua_state`，新的lua文件在首次加载的时候，会去读取文件加载代码，然后存放到lua的全局变量中，请求再次的时候 就会在`lua_state`全局table缓存中找到了，不需要再读取文件加载代码，因此修改完代码之后，需要`reload nginx`之后才可以生效。

3、通过`content_by_lua_file`中使用Nginx变量时，可以在实现在`lua_code_cache on`的情况下动态加载新的 Lua 脚本，而不需要reload nginx。



## OpenResty源码剖析-lua代码的执行

### **代码的执行** 

在`init_by_lua`等阶段openresty是在主协程中通过`lua_pcall`直接执行`lua`代码，而在`access_by_lua  content_by_lua`等阶段中，openresty创建一个新的协程，通过`lua_resume`执行lua代码，二者的区别在于能否执行`ngx.slepp. ngx.thread ngx.socket`这些有让出操作的函数

我们依旧以`content_by_**`阶段为例进行讲解

#### **content_by_\**阶段**

`content_by_**`阶段对应的请求来临时，执行流程为`ngx_http_lua_content_handler -> ngx_http_lua_content_handler_file-> ngx_http_lua_content_by_chunk`

`ngx_http_lua_content_handler`和`ngx_http_lua_content_handler_file`完成了请求上下文初始化，代码加载等操作`ngx_http_lua_content_by_chunk`进行代码的执行工作

#### **ngx_http_lua_content_by_chunk**

```c
ngx_int_t
ngx_http_lua_content_by_chunk(lua_State *L, ngx_http_request_t *r)
{
...
    ctx->entered_content_phase = 1;//标示当前进入了content_phase

    /*  {{{ new coroutine to handle request */
    co = ngx_http_lua_new_thread(r, L, &co_ref);//创建了一个新的lua协程

...
    /*  move code closure to new coroutine */
    lua_xmove(L, co, 1);//主协程的栈顶是需要执行的lua函数，通过lua_xmove将栈顶函数交换到新lua协程中

    /*  set closure's env table to new coroutine's globals table */
    ngx_http_lua_get_globals_table(co);
    lua_setfenv(co, -2);

    /*  save nginx request in coroutine globals table */
    ngx_http_lua_set_req(co, r);//把当前请求r赋值给新协程的全局变量中
...
    rc = ngx_http_lua_run_thread(L, r, ctx, 0);//运行新协程
...
    if (rc == NGX_AGAIN) {
        return ngx_http_lua_content_run_posted_threads(L, r, ctx, 0);//执行需要延后执行的协程，0表示上面传来的状态是NGX_AGAIN
    }

    if (rc == NGX_DONE) {
        return ngx_http_lua_content_run_posted_threads(L, r, ctx, 1);//执行需要延后执行的协程，1表示上面传来的状态是NGX_DONE
    }

    return NGX_OK;
}
```

27-50行，有一步是重新设置请求的上下文，将用于标示当前进入了那个阶段的变量重置为0

```c
     ctx->entered_rewrite_phase = 0;
     ctx->entered_access_phase = 0;
     ctx->entered_content_phase = 0;
```

这几个字段的用处在`ngx_http_lua_content_handler`函数中用于确认之前是否进入过对应阶段

```c
ngx_int_t
ngx_http_lua_content_handler(ngx_http_request_t *r)
{
...
    if (ctx->entered_content_phase) {
        dd("calling wev handler");
        rc = ctx->resume_handler(r);
        dd("wev handler returns %d", (int) rc);
        return rc;
    }
...
}
```

53行，创建了一个新的lua协程

63行，加载代码的时候，我们把需要执行的lua函数放到了主协程的栈顶，所以这里我们需要通过`lua_xmove`将函数移到新协程中

70行，把当前请求r赋值给新协程的全局变量中，从而可以让lua执行获取和请求相关的一些函数，比如`ngx.req.get_method()`和`ngx.set_method,ngx.req.stat_time()`等

103行，运行新创建的lua协程

109-114行，`ngx.thread.spawn`中创建子协程后，会调用`ngx_http_lua_post_thread`。`ngx_http_lua_post_thread`函数将父协程放在了`ctx->posted_threads`指向的链表中，这里的`ngx_http_lua_content_run_posted_threads`运行延后执行的主协程

#### ***ngx_http_lua_new_thread创建协程***

```c
lua_State *
ngx_http_lua_new_thread(ngx_http_request_t *r, lua_State *L, int *ref)
{
...
    base = lua_gettop(L);

    lua_pushlightuserdata(L, &ngx_http_lua_coroutines_key);//获取全局变量中储存协程的table
    lua_rawget(L, LUA_REGISTRYINDEX);

    co = lua_newthread(L);//创建新协程
...
    *ref = luaL_ref(L, -2);//将创建的新协程保存对应的全局变量中

    if (*ref == LUA_NOREF) {
        lua_settop(L, base);  /* restore main thread stack */
        return NULL;
    }

    lua_settop(L, base);//恢复主协程的栈空间大小
    return co;
}
```

312行，获得了主协程栈中有多少元素

314-315行，获得全局变量中储存协程的table  `LUA_REGISTRYINDEX[‘ngx_http_lua_code_coroutines_key’]`

因为lua中协程也是GC的对象，会被lua系统进行垃圾回收，为了保证挂起的协程不会被GC掉，openresty使用了 `LUA_REGISTRYINDEX[‘ngx_http_lua_code_coroutines_key’]`来保存新创建的协程，在协程执行完毕后将协程从table

中删除，使的GC可以将这个协程垃圾回收掉

317行，创建了一个`lua_newthread`并把其压入主协程的栈顶

334行，将新创建的协程保存到LUA_REGISTRYINDEX[‘ngx_http_lua_code_coroutines_key’]

341行，恢复主协程的栈空间大小

343行，返回新创建的协程

#### **ngx_http_lua_run_thread运行协程**

`ngx_http_lua_run_thread`函数的代码行数比较多，有500多行，内容如下：

```c
ngx_http_lua_run_thread(lua_State *L, ngx_http_request_t *r,
    ngx_http_lua_ctx_t *ctx, volatile int nrets)
{
...
    NGX_LUA_EXCEPTION_TRY {
...
        for ( ;; ) {
...
            orig_coctx = ctx->cur_co_ctx;
...
             rv = lua_resume(orig_coctx->co, nrets);//通过lua_resume执行协程中的函数
 ...
             switch (rv) {//处理lua_resume的返回值
             case LUA_YIELD:
 ..
                 if (r->uri_changed) {
                     return ngx_http_lua_handle_rewrite_jump(L, r, ctx);
                 }
                 if (ctx->exited) {
                     return ngx_http_lua_handle_exit(L, r, ctx);
                 }
                 if (ctx->exec_uri.len) {
                     return ngx_http_lua_handle_exec(L, r, ctx);
                 }
                 switch(ctx->co_op) {
 ...
                 }
                 continue;
             case 0:
 ...
                 continue;
 ...
             default:
                 err = "unknown error";
                 break;
             }
 ...
         }
     } NGX_LUA_EXCEPTION_CATCH {
         dd("nginx execution restored");
     }
     return NGX_ERROR;

 no_parent:
 ...
     return (r->header_sent || ctx->header_sent) ?
                 NGX_ERROR : NGX_HTTP_INTERNAL_SERVER_ERROR;

 done:
 ...
     return NGX_OK;
 }
```

1015行，通过`lua_resume`执行协程的函数，并根据返回的结果进行不同的处理

`LUA_YIELD`: 协程被挂起

`0:` 协程执行结束

其他: 运行出错，如内存不足等

```c
             switch (rv) {
             case LUA_YIELD:
 ...
                 if (r->uri_changed) {
                     return ngx_http_lua_handle_rewrite_jump(L, r, ctx);//调用了ngx.redirect
                 }

                 if (ctx->exited) {
                     return ngx_http_lua_handle_exit(L, r, ctx);//调用了ngx.exit
                 }

                 if (ctx->exec_uri.len) {
                     return ngx_http_lua_handle_exec(L, r, ctx);//调用了ngx.exec
                 }   
```

**lua_resume返回LUA_YIELD，表示被挂起**

先处理以下3种情况：

`r->uri_changed`为true表明调用了`ngx.redirect`

`ext->exited`为true表明调用了`ngx.exit`

`ctx->exec_uri.len`为true表明调用了`ngx.exec`

其余情况需要再比较`ctx->co_op`的返回值

```c
                 switch(ctx->co_op) {
                 case NGX_HTTP_LUA_USER_CORO_NOP:
 ...
                     ctx->cur_co_ctx = NULL;
                     return NGX_AGAIN;
                 case NGX_HTTP_LUA_USER_THREAD_RESUME://ngx.thread.spawn
 ...
                     ctx->co_op = NGX_HTTP_LUA_USER_CORO_NOP;
                     nrets = lua_gettop(ctx->cur_co_ctx->co) - 1;
                     dd("nrets = %d", nrets);
 ...
                     break;
                 case NGX_HTTP_LUA_USER_CORO_RESUME://coroutine.resume
 ...
                     ctx->co_op = NGX_HTTP_LUA_USER_CORO_NOP;
                     old_co = ctx->cur_co_ctx->parent_co_ctx->co;
                     nrets = lua_gettop(old_co);
                     if (nrets) {
                         dd("moving %d return values to parent", nrets);
                         lua_xmove(old_co, ctx->cur_co_ctx->co, nrets);
 ...
                     }
                     break;
                 default://coroutine.yield 
```

在openresty内部重新实现的`coroutine.yield` 和`coroutine.resume` 和`ngx.thread.spawn`中 会对`ctx->co_op`进行赋值

1064行，`case NGX_HTTP_LUA_USER_CORO_NOP`表示不再有协程需要处理了，跳出这一次循环，等待下一次的读写时间，或者定时器到期

1071行，`case NGX_HTTP_USER_THREAD_RESUME`对应`ngx.thread.spawn`被调用的情况

1085行，`case NGX_HTTP_LUA_CORO_RESUME` 对应有lua代码调用`coroutine.resume`，把当前线程标记为`NGX_HTTP_LUA_USER_CORO_NOP`

1106行，`default` 对应`NGX_HTTP_LUA_CODO_YIELD`，对应`coroutine.yield`被调用的情况

```c
                 default:
 ...
                    ctx->co_op = NGX_HTTP_LUA_USER_CORO_NOP;

                    if (ngx_http_lua_is_thread(ctx)) {
...
                        ngx_http_lua_probe_info("set co running");
                        ctx->cur_co_ctx->co_status = NGX_HTTP_LUA_CO_RUNNING;

                        if (ctx->posted_threads) {
                            ngx_http_lua_post_thread(r, ctx, ctx->cur_co_ctx);
                            ctx->cur_co_ctx = NULL;
                            return NGX_AGAIN;
                        }
...
                        nrets = 0;
                        continue;
                    }
...
                    nrets = lua_gettop(ctx->cur_co_ctx->co);
                    next_coctx = ctx->cur_co_ctx->parent_co_ctx;
                    next_co = next_coctx->co;
...
                    lua_pushboolean(next_co, 1);

                    if (nrets) {
                        dd("moving %d return values to next co", nrets);
                        lua_xmove(ctx->cur_co_ctx->co, next_co, nrets);
                    }
                    nrets++;  /* add the true boolean value */
                    ctx->cur_co_ctx = next_coctx;
                    break;
                } 
```

`default` 对应`NGX_HTTP_LUA_CODO_YIELD`,表示`coroutine.yield`被调用的情况

1121行，判断是不是主协程，或者是调用`ngx.thread.spawn`的协程

1135行，判断链表中有没有排队需要执行的协程，如果有的话，调用`ngx_http_lua_post_thread`将这个协程放到他们的后面，没有的话，直接让他自己恢复执行即可，回到`for`循环开头

1136-1167行，`ngx.thread.spawn`创建的子协程,需要将返回值放入父协程中

1150-1152行和1165行，将当前需要执行的协程，由子协程切换为父协程

1159行，放入布尔值`true`

1161行，将子协程的所有返回值通过`lua_xmove`放入父协程中

1170行，由于多了一个布尔值`true`返回值个数`+1`

1166行，回到`for`循环开头，在父协程上执行`lua_resume`

**lua_resume返回0，表示当前协程执行完毕**

这里因为有`ngx.thread API`的存在，可能有多个协程在跑，需要判断父协程和所有的子协程的运行情况。

```c
            case 0:
...
                if (ngx_http_lua_is_entry_thread(ctx)) {
...
                    ngx_http_lua_del_thread(r, L, ctx, ctx->cur_co_ctx);
                    if (ctx->uthreads) {
                        ctx->cur_co_ctx = NULL;
                        return NGX_AGAIN;
                    }
                    /* all user threads terminated already */
                    goto done;
                }
                if (ctx->cur_co_ctx->is_uthread) {
...
                    ngx_http_lua_del_thread(r, L, ctx, ctx->cur_co_ctx);
                    ctx->uthreads--;
                    if (ctx->uthreads == 0) {
                        if (ngx_http_lua_entry_thread_alive(ctx)) {
                            ctx->cur_co_ctx = NULL;
                            return NGX_AGAIN;
                        }
                        goto done;
                    }
                    /* some other user threads still running */
                    ctx->cur_co_ctx = NULL;
                    return NGX_AGAIN;
                }
```

1183行，判断是不是主协程

1187行，执行完毕的协程是主协程，从全局table中删除这个协程

1188-1193行，判断还在运行的子协程个数，如果非0返回`NGX_AGAIN`,否则`goto done`进行一些数据发送的相关工作并返回`NGX_OK`

1195-1233，判断执行完毕的是不是子协程

1223行，由于协程已经执行完毕，从全局`table`中删除这个协程，可以被`lua  GC`掉

1223行，还在运行的子协程个数`-1`

1226行，判断主协程是否还需要运行，是的话，返回`NGX_AGAIN`，否则`goto done`，进行一些数据发送的相关工作并返回`NGX_OK`

1232-1234行，表示有子协程还在运行，返回`NGX_AGAIN`

### **总结**

1、在`init_by_lua`等阶段 ，openresty是在主协程中通过`lua_pcall`直接执行lua代码，而在`access_by_lua、content_by_lua`等阶段中，openresty创建一个新的协程，通过`lua_resume`执行lua代码

2、openresty将要延后执行的协程放入链表中，在`*_run_posted_threads`函数中通过调用`ngx_http_lua_run_thread`进行执行。