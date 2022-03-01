---
title: "nginx源码分析-模块化架构"
date: 2022-03-01T18:34:14+08:00
tags:
   - nginx源码分析 
   - nginx 
categories:
   - nginx源码分析 
toc: true
---

> 从源码角度分析一下nginx的模块化设计架构，主要通过nginx-1.15.8源码以及陶辉写的《深入理解Nginx模块开发与架构解析》进行分析的。

## nginx模块的数据结构

### nginx模块化的设计主要体现在`ngx_module_t`的数据结构

```c
struct ngx_module_s {
    ngx_uint_t            ctx_index;
    ngx_uint_t            index;

    char                 *name;

    ngx_uint_t            spare0;
    ngx_uint_t            spare1;

    ngx_uint_t            version;
    const char           *signature;

    void                 *ctx;
    ngx_command_t        *commands;
    ngx_uint_t            type;

    ngx_int_t           (*init_master)(ngx_log_t *log);

    ngx_int_t           (*init_module)(ngx_cycle_t *cycle);

    ngx_int_t           (*init_process)(ngx_cycle_t *cycle);
    ngx_int_t           (*init_thread)(ngx_cycle_t *cycle);
    void                (*exit_thread)(ngx_cycle_t *cycle);
    void                (*exit_process)(ngx_cycle_t *cycle);

    void                (*exit_master)(ngx_cycle_t *cycle);

    uintptr_t             spare_hook0;
    uintptr_t             spare_hook1;
    uintptr_t             spare_hook2;
    uintptr_t             spare_hook3;
    uintptr_t             spare_hook4;
    uintptr_t             spare_hook5;
    uintptr_t             spare_hook6;
    uintptr_t             spare_hook7;
};

typedef struct ngx_module_s          ngx_module_t;
```
#### 主要字段说明

- `ctx_index`表示当前模块在这类(`type`类型)模块中的序号。它非常重要，Nginx的模块化设计非常依赖于各个模块的顺序，它们即用于表达优先级，也用于表明每个模块的位置，以便nginx框架快速获得某个模块的数据。

`ctx_index`赋值主要在`ngx_count_modules`函数处理的。

- `index`表示当前模块在nginx所有模块`ngx_modules`中的序号，nginx在启动时会根据`ngx_modules`数组设置各个模块的`index`值
```c
ngx_int_t
ngx_preinit_modules(void)
{
    ngx_uint_t  i;

    for (i = 0; ngx_modules[i]; i++) {
        ngx_modules[i]->index = i;
        ngx_modules[i]->name = ngx_module_names[i];
    }

    ngx_modules_n = i;
    ngx_max_module = ngx_modules_n + NGX_MAX_DYNAMIC_MODULES;

    return NGX_OK;
}
```

- `ctx`

`ctx`用于指向一类模块的上下文结构体，指向这类特定模块的公共接口，只对HTTP模块，主要`ngx_http_module_t`结构体，后面会介绍这个结构体

- `commands`

模块所支持的指令。

- `type`

模块的类型, 与`ctx`密切相关，主要有这种类型`NGX_HTTP_MODULE、NGX_CORE_MODULE、NGX_CONF_MODULE、NGX_EVENT_MODULE、NGX_MAIL_MODULE、NGX_STREAM_MODULE`

- `init_master`

代码中好像没有调用，貌似做保留用，从字面上来，应该是master进程启动时调用。

- `init_module`

每个模块如果设置这个回调函数，初始化时调用。
```c
ngx_int_t
ngx_init_modules(ngx_cycle_t *cycle)
{
    ngx_uint_t  i;

    for (i = 0; cycle->modules[i]; i++) {
        if (cycle->modules[i]->init_module) {
            if (cycle->modules[i]->init_module(cycle) != NGX_OK) {
                return NGX_ERROR;
            }
        }
    }

    return NGX_OK;
}
```

`ngx_init_modules`函数主要是由master进程fork子进程前的`ngx_init_cycle`调用

- `init_process`

子进程for完成后调用。主要由`ngx_worker_process_init`调用，而`ngx_worker_process_init`主要有fork子进程后的`ngx_worker_process_cycle`调用。

`ngx_master_process_cycle`-->`ngx_start_worker_processes`-->`ngx_worker_process_cycle`-->`ngx_worker_process_init`-->`cycle->modules[i]->init_process(cycle)`

- `init_thread`

多线程模式下调用，nginx已经支持多线程模式，但是再这里暂不讨论。

- `exit_thread`

多线程模式下调用，nginx已经支持多线程模式，但是再这里暂不讨论。

- `exit_process`

woker子进程时调用。

- `exit_master`

master进程时调用。

模块的数据结构中的初始化回调函数`init_module、init_process、exit_process、exit_master`是由nginx框架调用的，跟HTTP框架无关，因此HTTP模块时可以设置为`NULL`，HTTP模块主要设置`ctx`和`commands`,
针对HTTP模块来说，`ctx`指向`ngx_http_moduel_t`的数据结构，`commands`被赋值于`ngx_command_t`的结构体。我们通过[lua-nginx-module](https://github.com/openresty/lua-nginx-module)的模块代码。
```c
ngx_module_t ngx_http_lua_module = {
    NGX_MODULE_V1,                                       // ngx_module_t数据结构的前7个字段，通过一个宏定义统一赋值
    &ngx_http_lua_module_ctx,   /*  module context */    //ctx字段, 主要是ngx_http_module_t指针
    ngx_http_lua_cmds,          /*  module directives */ //commands字段, 只要是ngx_command_t结构体
    NGX_HTTP_MODULE,            /*  module type */       //type字段
    NULL,                       /*  init master */       //init_master回调函数
    NULL,                       /*  init module */       //init_module回调函数
    ngx_http_lua_init_worker,   /*  init process */      //init_process回调函数
    NULL,                       /*  init thread */       //init_thread回调函数 
    NULL,                       /*  exit thread */       //exit_thread回调函数
    NULL,                       /*  exit process */      //exit_process回调函数
    NULL,                       /*  exit master */       //exit_master回调函数
    NGX_MODULE_V1_PADDING                                //其他字段剩余字段统一宏定义赋值
};
```

`ctx`的数据结构在`ngx_module_t`中是`void`类型，可以是任何指针类型，在HTTP模块主要是`ngx_http_module_t`的数据结构指针，其他模块可能是其他类型，我们先从HTTP模块分析

### HTTP模块的ctx数据结构
HTTP模块的ctx主要是`ngx_http_module_t`的结构体指针，我们来看一下这个结构体
```c
typedef struct {
    ngx_int_t   (*preconfiguration)(ngx_conf_t *cf);                        //解析配置前调用
    ngx_int_t   (*postconfiguration)(ngx_conf_t *cf);                       //完成配置解析后调用

    void       *(*create_main_conf)(ngx_conf_t *cf);                        //main级别(http{}块配置项)中的全局配置时，回调函数创建存储全局配置的结构体 
    char       *(*init_main_conf)(ngx_conf_t *cf, void *conf);              //main级别配置项的初始化

    void       *(*create_srv_conf)(ngx_conf_t *cf);                         //srv级别(server{}块)的配置时，回调函数创建存储配置的结构体
    char       *(*merge_srv_conf)(ngx_conf_t *cf, void *prev, void *conf);  //srv级别初始化回调函数

    void       *(*create_loc_conf)(ngx_conf_t *cf);                         //loc级别(location{})的配置时，回调函数创建存储配置的结构体
    char       *(*merge_loc_conf)(ngx_conf_t *cf, void *prev, void *conf);  //用于合并srv级别和loc级别同名配置项
} ngx_http_module_t;
```
这8个回调函数在HTTP模块的调用顺序可能跟定义顺序不一致，实际顺序应该如下：

1. create_main_conf

2. create_srv_conf

3. creat_loc_conf

4. preconfiguration

5. init_main_conf

6. init_srv_conf

7. init_loc_conf

8. postconfiguration

这8个函数都在`ngx_http_block`函数中调用
```c
static char *
ngx_http_block(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    ...
    //初始化ngx_http_conf_ctx_t的ctx，ctx->main_conf，ctx->srv_conf，ctx->loc_conf

    //调用逐个Http模块的create_main_conf，create_srv_conf，create_loc_conf

    //调用preconfiguration

    //调用http{}块的配置
    rv = ngx_conf_parse(cf, NULL);

    //调用各个模块的init_main_conf，init_srv_conf，init_loc_conf

    //调用各个模块的postconfiguration

    ...
}
```

我们来看一下[lua-nginx-module](https://github.com/openresty/lua-nginx-module)的`ngx_http_module_t`
```c
ngx_http_module_t ngx_http_lua_module_ctx = {
    NULL,                             /*  preconfiguration */
    ngx_http_lua_init,                /*  postconfiguration */

    ngx_http_lua_create_main_conf,    /*  create main configuration */
    ngx_http_lua_init_main_conf,      /*  init main configuration */

    ngx_http_lua_create_srv_conf,     /*  create server configuration */
    ngx_http_lua_merge_srv_conf,      /*  merge server configuration */

    ngx_http_lua_create_loc_conf,     /*  create location configuration */
    ngx_http_lua_merge_loc_conf       /*  merge location configuration */
};
```
除了`preconfiguration`为`NULL`，其他都赋值了相应的回调函数。

### 指令数据结构
指令用于定义模块的配置文件的指令参数，主要是`ngx_command_t`的数据结构
```c
struct ngx_command_s {
    ngx_str_t             name;   //指令名称，比如gzip
    ngx_uint_t            type;   //类型，出现的位置，比如http{}，server{}, loc{}
    char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);  //解析指令的回调函数
    ngx_uint_t            conf; //在配置文件中偏移量
    ngx_uint_t            offset; //通常用于预设的解析方法配置项，这是配置模块的一个优秀设计，需要conf配合使用
    void                 *post;   //配置读取后的处理方法
};

typedef struct ngx_command_s         ngx_command_t;
```

我们来看一下[lua-nginx-module](https://github.com/openresty/lua-nginx-module)的`ngx_command_t`，这个模块的指令太多，我截取一部分看一下
```c
static ngx_command_t ngx_http_lua_cmds[] = {
    ...
    /* access_by_lua "<inline script>" */
    { ngx_string("access_by_lua"),
      NGX_HTTP_MAIN_CONF|NGX_HTTP_SRV_CONF|NGX_HTTP_LOC_CONF|NGX_HTTP_LIF_CONF
                        |NGX_CONF_TAKE1,
      ngx_http_lua_access_by_lua,
      NGX_HTTP_LOC_CONF_OFFSET,
      0,
      (void *) ngx_http_lua_access_handler_inline },

    /* access_by_lua_block { <inline script> } */
    { ngx_string("access_by_lua_block"),
      NGX_HTTP_MAIN_CONF|NGX_HTTP_SRV_CONF|NGX_HTTP_LOC_CONF|NGX_HTTP_LIF_CONF
                        |NGX_CONF_BLOCK|NGX_CONF_NOARGS,
      ngx_http_lua_access_by_lua_block,
      NGX_HTTP_LOC_CONF_OFFSET,
      0,
      (void *) ngx_http_lua_access_handler_inline },
    ...
    ngx_null_command
}
```

指令`accee_by_lua`的set函数`ngx_http_lua_access_by_lua`，post函数`ngx_http_lua_access_handler_inline`

## 总结
开发nginx第三方模块时，要严格`ngx_module_t`格式定义模块，只有这样nginx框架才能调用这个三方模块。
