---
title: "lua-nginx-module模块源码浅析"
date: 2022-03-03T20:07:55+08:00
tags:
   - nginx源码分析
   - lua-nginx
categories:
   - nginx源码分析
toc: true
---

> 最近在公司接入层的nginx支持QUIC+HTTP3的工作，我们的nginx用了[lua-nginx-module](https://github.com/openresty/lua-nginx-module)模块，在启用HTTP3时，这个模块偶发`epoll_ctl(1, 3) failed (17: File exists)`的错误，
> 感觉应该是lua-nginx-module对HTTP3支持不好，因此看一下源码，适配一下HTTP3。

## 架构
学习lua-nginx-module模块前，先了解这个模块指令在nginx处理阶段的执行调用的情况，再次借用一下官方的图。
![01](./lua-nginx-arch.png)

```c
  //master进程初始化时调用
  init_by_lua*

  //worker进程初始化时调用
  init_worker_by_lua* 

  //ssl握手阶段调用
  ssl_certificate_by_lua*

  set_by_lua*

  //rewrite阶段调用
  rewrite_by_lua*

  //access阶段调用
  access_by_lua*

  //content阶段调用
  content_by_lua*

  //设置upstream阶段调用
  balancer_by_lua*

  //过滤header头
  header_filter_by_lua*

  //过滤body
  body_filter_by_lua*

  //日志阶段
  log_by_lua*
```

## 模块初始化
1. lua-nginx-module也是nginx的第三方模块，它也遵守nginx模块开发规范，可以查看[nginx模块化架构](https://lizj3624.github.io/post/ngx-module-arch/)了解nginx的模块化开发，nginx加载lua指令时会有初始化。
先看一下这个模块的定义:
```c
ngx_module_t ngx_http_lua_module = {
    NGX_MODULE_V1,
    &ngx_http_lua_module_ctx,   /*  module context */
    ngx_http_lua_cmds,          /*  module directives */
    NGX_HTTP_MODULE,            /*  module type */
    NULL,                       /*  init master */
    NULL,                       /*  init module */
    ngx_http_lua_init_worker,   /*  init process */
    NULL,                       /*  init thread */
    NULL,                       /*  exit thread */
    NULL,                       /*  exit process */
    NULL,                       /*  exit master */
    NGX_MODULE_V1_PADDING
};
```

lua模块的init_process回调函数
```c
ngx_int_t
ngx_http_lua_init_worker(ngx_cycle_t *cycle)
{
    //获取lua模块的配置信息
    lmcf = ngx_http_cycle_get_module_main_conf(cycle, ngx_http_lua_module);
    ...
    http_ctx
    ...
    //模块create_srv_conf，merge_srv_conf，create_loc_conf，merge_loc_conf
    ...
    ctx = ngx_http_lua_create_ctx(r);
    ...

    //http request放到lua中
    ngx_http_lua_set_req(lmcf->lua, r);

    //init_worker_by的post回调函数
    (void) lmcf->init_worker_handler(cycle->log, lmcf, lmcf->lua);

    ...
}
```

> ngx_conf_parse====>ngx_conf_handler====>>ngx_http_block====>postconfiguration

lua模块的postconfiguration回调函数，初始化lua vm
```c
static ngx_int_t
ngx_http_lua_init(ngx_conf_t *cf)
{
    ...
    lmcf = ngx_http_conf_get_module_main_conf(cf, ngx_http_lua_module);
    ...
    //设置ngx http处理阶段的回调函数
    cmcf = ngx_http_conf_get_module_main_conf(cf, ngx_http_core_module);

    if (lmcf->requires_rewrite) {
        h = ngx_array_push(&cmcf->phases[NGX_HTTP_REWRITE_PHASE].handlers);
        if (h == NULL) {
            return NGX_ERROR;
        }

        *h = ngx_http_lua_rewrite_handler;
    }

    if (lmcf->requires_access) {
        h = ngx_array_push(&cmcf->phases[NGX_HTTP_ACCESS_PHASE].handlers);
        if (h == NULL) {
            return NGX_ERROR;
        }

        *h = ngx_http_lua_access_handler;
    }

    dd("requires log: %d", (int) lmcf->requires_log);

    if (lmcf->requires_log) {
        arr = &cmcf->phases[NGX_HTTP_LOG_PHASE].handlers;
        h = ngx_array_push(arr);
        if (h == NULL) {
            return NGX_ERROR;
        }

        if (arr->nelts > 1) {
            h = arr->elts;
            ngx_memmove(&h[1], h,
                        (arr->nelts - 1) * sizeof(ngx_http_handler_pt));
        }

        *h = ngx_http_lua_log_handler;
    }

    if (multi_http_blocks || lmcf->requires_header_filter) {
        rc = ngx_http_lua_header_filter_init();
        if (rc != NGX_OK) {
            return rc;
        }
    }

    if (multi_http_blocks || lmcf->requires_body_filter) {
        rc = ngx_http_lua_body_filter_init();
        if (rc != NGX_OK) {
            return rc;
        }
    }

    ...
    //如果lua环境不存在，初始化
    if (lmcf->lua == NULL) {
        lmcf->lua = ngx_http_lua_init_vm(NULL, cf->cycle, cf->pool, lmcf,
                                         cf->log, NULL);
        ...
        //init_by_lua的post回调函数ngx_http_lua_init_by_file
        rc = lmcf->init_handler(cf->log, lmcf, lmcf->lua);
        ...
    }
}
```

2. lua-nginx-module这个模块赋予了nginx支持lua的能力，这个模块在nginx启动时初始化lua的执行环境。

为worker初始化lua vm函数，lua_State
```c
static lua_State *
ngx_http_lua_new_state(lua_State *parent_vm, ngx_cycle_t *cycle,
    ngx_http_lua_main_conf_t *lmcf, ngx_log_t *log)
{
    ...
    L = luaL_newstate();
    ...
    ngx_http_lua_init_registry(L, log);
    ngx_http_lua_init_globals(L, cycle, lmcf, log);

    return L;
}
```

## 模块指令的执行过程

以`access_by_lua*`这类指令执行的过程，看一下指令的指令流程, 先看一下指令的定义
```c
    { ngx_string("access_by_lua_file"),
      NGX_HTTP_MAIN_CONF|NGX_HTTP_SRV_CONF|NGX_HTTP_LOC_CONF|NGX_HTTP_LIF_CONF
                        |NGX_CONF_TAKE1,
      ngx_http_lua_access_by_lua,
      NGX_HTTP_LOC_CONF_OFFSET,
      0,
      (void *) ngx_http_lua_access_handler_file },
```

指令的set函数`ngx_http_lua_access_by_lua`, post函数是`ngx_http_lua_access_handler_file`

```c
char *
ngx_http_lua_access_by_lua(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
    //加载lua文件
    //设置access_handler回调, post函数(ngx_http_lua_access_handler_file)
    llcf->access_handler = (ngx_http_handler_pt) cmd->post;
}

ngx_int_t
ngx_http_lua_access_handler_file(ngx_http_request_t *r)
{
    ...
    //获取lua vm
    L = ngx_http_lua_get_lua_vm(r, NULL);

    //加载lua到cache中
    /*  load Lua script file (w/ cache)        sp = 1 */
    rc = ngx_http_lua_cache_loadfile(r->connection->log, L, script_path,
                                     llcf->access_src_key);
    ...
    return ngx_http_lua_access_by_chunk(L, r);
}

static ngx_int_t
ngx_http_lua_access_by_chunk(lua_State *L, ngx_http_request_t *r)
{
    //开启lua协程,为每个请求分配一个协程
    co = ngx_http_lua_new_thread(r, L, &co_ref);

    ...
    //将request请求与协程绑定
    ngx_http_lua_set_req(co, r);

    ...
    //执行协程
    rc = ngx_http_lua_run_thread(L, r, ctx, 0);

    if (rc == NGX_AGAIN) {
        //协程处理
        rc = ngx_http_lua_run_posted_threads(c, L, r, ctx, nreqs);

        if (rc == NGX_ERROR || rc == NGX_DONE || rc > NGX_OK) {
            return rc;
        }

        if (rc != NGX_OK) {
            return NGX_DECLINED;
        }

    } else if (rc == NGX_DONE) {
        //释放协程
        ngx_http_lua_finalize_request(r, NGX_DONE);

        rc = ngx_http_lua_run_posted_threads(c, L, r, ctx, nreqs);

        if (rc == NGX_ERROR || rc == NGX_DONE || rc > NGX_OK) {
            return rc;
        }

        if (rc != NGX_OK) {
            return NGX_DECLINED;
        }
    }
    ...
}
```

## 总结
从源码大致看一下lua-nginx-module的执行过程，lua vm初始化部分还有一些疑惑，后期再细细看看。
