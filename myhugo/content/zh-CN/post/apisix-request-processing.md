---
title: "APISIX源码分析-请求处理流程"
date: 2022-04-05T10:33:34+08:00
tags:
   - apisix
   - nginx
categories:
   - apisix
   - nginx
toc: true
---

APISIX的请求处理流程跟Nginx处理请求的各个阶段是一致的，只是APISIX在各个阶段嵌入自己详细处理请求的逻辑。

## init_worker阶段 
`init_worker`阶段主要是在Nginx的worker初始化阶段，虽然不参与请求的处理，但是这个节点做了很多准备工作，针对APISIX来说这阶段从etcd获取
`route、server、plugin、upstream`信息，并缓存在每个worker中，这个节点主要函数`http_init_worker`。
```conf
init_worker_by_lua_block {
    apisix.http_init_worker()
}
```
看一下这个[函数](https://github.com/apache/apisix/blob/7e1babafd098e26f2648d5af3b7695b3020290db/apisix/init.lua#L92)主要处理逻辑，这个函数主要在`init.lua`源文件中
```lua
function _M.http_init_worker()
    ...
    -- worker事件，如果路由、上游等信息变更通知，这个机制是跟kong一致
    local we = require("resty.worker.events")
    local ok, err = we.configure({shm = "worker-events", interval = 0.1})
    if not ok then
        error("failed to init worker event: " .. err)
    end

    -- 服务发现
    local discovery = require("apisix.discovery.init").discovery
    if discovery and discovery.init_worker then
        discovery.init_worker()
    end

    -- upstream负载均衡器的init_worker
    require("apisix.balancer").init_worker()
    load_balancer = require("apisix.balancer")
    require("apisix.admin.init").init_worker()

    require("apisix.timers").init_worker()

    require("apisix.debug").init_worker()

    -- 插件init_worker初始化
    plugin.init_worker()

    -- http路由的init_worker初始化
    router.http_init_worker()
    require("apisix.http.service").init_worker()
    plugin_config.init_worker()
    require("apisix.consumer").init_worker()
    ...
    -- upstream的init_worker初始化
    apisix_upstream.init_worker()
    require("apisix.plugins.ext-plugin.init").init_worker()
    ...
 end
```
这些`init_worker`函数主要是从配置中跟key前缀拉取数据，并缓存在worker中，看一下七层HTTP路由函数`router.http_init_worker()`
```lua
function _M.http_init_worker()
    local conf = core.config.local_conf()
    local router_http_name = "radixtree_uri"
    local router_ssl_name = "radixtree_sni"

    -- 路由匹配支持三种模式，radixtree_uri, radixtree_host_uri，radixtree_uri_with_paramter，都是基于基数树。 
    -- 从配置config-default.yaml获取路由匹配模式. 
    if conf and conf.apisix and conf.apisix.router then
        router_http_name = conf.apisix.router.http or router_http_name
        router_ssl_name = conf.apisix.router.ssl or router_ssl_name
    end

    -- 根据路由匹配模式获取引用的处理模块，相应的模块主要在apisix/http/route模块下
    local router_http = require("apisix.http.router." .. router_http_name)
    attach_http_router_common_methods(router_http)

    -- 主要调用时apisix/http/route.lua中init_worker函数
    router_http.init_worker(filter)
    _M.router_http = router_http

    -- 处理ssl的路由
    local router_ssl = require("apisix.ssl.router." .. router_ssl_name)
    router_ssl.init_worker()
    _M.router_ssl = router_ssl

    _M.api = require("apisix.api_router")

    local global_rules, err = core.config.new("/global_rules", {
            automatic = true,
            item_schema = core.schema.global_rule,
            checker = plugin_checker,
        })
    if not global_rules then
        error("failed to create etcd instance for fetching /global_rules : "
              .. err)
    end
    _M.global_rules = global_rules
end
```
我们在跟进看一下`route.lua`的`init_worker`函数
```lua
function _M.init_worker(filter)
    local user_routes, err = core.config.new("/routes", {
            automatic = true,
            item_schema = core.schema.route,
            checker = check_route,
            filter = filter,
        })
    if not user_routes then
        error("failed to create etcd instance for fetching /routes : " .. err)
    end

    return user_routes
end
```
这里的`core.config.new`是调用的配置中心`config_etcd.lua`的`new`函数。
```lua
-- core.lua

-- 根据配置config_default.yaml的config_center选择配置中心，默认是etcd
local config_center = local_conf.apisix and local_conf.apisix.config_center
                      or "etcd"
log.info("use config_center: ", config_center)

-- config就是选择配置中心的处理模块，在core/config_*.lua
local config = require("apisix.core.config_" .. config_center)
config.type = config_center
```

在先看一下`core/config_etcd.lua`的`new`函数
```lua
function _M.new(key, opts)
    local local_conf, err = config_local.local_conf()
    if not local_conf then
        return nil, err
    end

    local etcd_conf = local_conf.etcd
    local prefix = etcd_conf.prefix
    local resync_delay = etcd_conf.resync_delay
    if not resync_delay or resync_delay < 0 then
        resync_delay = 5
    end
    local health_check_timeout = etcd_conf.health_check_timeout
    if not health_check_timeout or health_check_timeout < 0 then
        health_check_timeout = 10
    end

    local automatic = opts and opts.automatic
    local item_schema = opts and opts.item_schema
    local filter_fun = opts and opts.filter
    local timeout = opts and opts.timeout
    local single_item = opts and opts.single_item
    local checker = opts and opts.checker

    local obj = setmetatable({
        etcd_cli = nil,
        key = key and prefix .. key,
        automatic = automatic,
        item_schema = item_schema,
        checker = checker,
        sync_times = 0,
        running = true,
        conf_version = 0,
        values = nil,
        need_reload = true,
        routes_hash = nil,
        prev_index = 0,
        last_err = nil,
        last_err_time = nil,
        resync_delay = resync_delay,
        health_check_timeout = health_check_timeout,
        timeout = timeout,
        single_item = single_item,
        filter = filter_fun,
    }, mt)

    if automatic then
        if not key then
            return nil, "missing `key` argument"
        end

        if loaded_configuration[key] then
            local res = loaded_configuration[key]
            loaded_configuration[key] = nil -- tried to load

            log.notice("use loaded configuration ", key)

            local dir_res, headers = res.body, res.headers
            load_full_data(obj, dir_res, headers)
        end

        -- 设置定时器的回调函数，定期从etcd更新数据
        ngx_timer_at(0, _automatic_fetch, obj)

    else
        local etcd_cli, err = get_etcd()
        if not etcd_cli then
            return nil, "failed to start a etcd instance: " .. err
        end
        obj.etcd_cli = etcd_cli
    end

    if key then
        created_obj[key] = obj
    end

    return obj
end
```

## SSL握手阶段
这个阶段设置的是`http_ssl_phase`函数，校验证书，支持动态更新证书和私钥
```lua
function _M.http_ssl_phase()
    local ngx_ctx = ngx.ctx
    local api_ctx = ngx_ctx.api_ctx

    if api_ctx == nil then
        api_ctx = core.tablepool.fetch("api_ctx", 0, 32)
        ngx_ctx.api_ctx = api_ctx
    end

    local ok, err = router.router_ssl.match_and_set(api_ctx)
    if not ok then
        if err then
            core.log.error("failed to fetch ssl config: ", err)
        end
        ngx_exit(-1)
    end
end


-- apisix/ssl/route/radixtree_sni.lua
function _M.match_and_set(api_ctx)
    -- redixtree route不存在时或者版本有变更时，重建 

    -- sni 
    sni, err = apisix_ssl.server_name()

    -- 查找路由
    local ok = radixtree_router:dispatch(sni_rev, nil, api_ctx)

    -- 更加sni设置证书和私钥
    ok, err = set_pem_ssl_key(sni, matched_ssl.value.cert,
                          matched_ssl.value.key)
    ...
end
```

## access阶段
这个阶段设置是`http_access_phase`，这个函数是APISIX处理请求的核心函数入口，这里匹配路由，处理插件，查找上游(upstream)，根据`balancer`挑选合适的上游server。

```lua
function _M.http_access_phase()
    -- 通过ngx.ctx在请求各阶段传递数据
    local ngx_ctx = ngx.ctx

    -- api_ctx是apisix请求上下文
    local api_ctx = core.tablepool.fetch("api_ctx", 0, 32)
    ngx_ctx.api_ctx = api_ctx

    -- 将ngx.var设置到api_ctx
    core.ctx.set_vars_meta(api_ctx)

    -- 路由匹配，apisix/http/route/radixtree_uri.lua的match函数，然后再调用apisix/http/route.lua的match_uri函数
    router.router_http.match(api_ctx)

    -- 根据匹配的路由查找插件、service、upstream
    local upstream = get_upstream_by_id(up_id)

    -- 挑选upstream的server
    local server, err = load_balancer.pick_server(route, api_ctx)

    ...
end
```
1. 路由匹配主要是[lua-resty-radixtree](https://github.com/api7/lua-resty-radixtree)，这是支流科技开源的基数树的实现，这个公司也是apisix的主要维护公司，是一家开源的商业公司。

2. 根据`balancer`挑选server主要是调用`balancer.lua`的`pick_server`函数

```lua
local function pick_server(route, ctx)
    ...
    -- 获取缓存数据
    local server_picker = ctx.server_picker
    if not server_picker then
        server_picker = lrucache_server_picker(key, version,
                                               create_server_picker, up_conf, checker)
    end
    if not server_picker then
        return nil, "failed to fetch server picker"
    end

    -- 根据负载算法获取一个合适的upstream server, apisix/balancer/*.lua是apisix支持的负载算法的实现
    local server, err = server_picker.get(ctx)
    if not server then
        err = err or "no valid upstream node"
        return nil, "failed to find valid upstream server, " .. err
    end
    ctx.balancer_server = server

    ...
end

_M.pick_server = pick_server
```
看一下`server_picker.get`函数如何根据负载算法挑选server的。优先看一下如果创建一个`server_picker`对象，这个实现主要在`create_server_picker`函数中
```lua
-- balancer.lua
local function create_server_picker(upstream, checker)
    local picker = pickers[upstream.type]
    if not picker then
        -- 根据upstream.type类型也就是负载算法(roundrobin\chash\ewma\least_conn)选择合适的负载算法模块(apisix/balancer/*.lua)
        pickers[upstream.type] = require("apisix.balancer." .. upstream.type)
        picker = pickers[upstream.type]
    end

    ...
    -- 调用相应负载算法模块的new方法
    local server_picker = picker.new(up_nodes[up_nodes._priority_index[1]], upstream)
    ...
end
```
然后在看一下具体一个负载算法的实现，那就以roundrobin为例。
```lua
local roundrobin  = require("resty.roundrobin")

function _M.new(up_nodes, upstream)
    local safe_limit = 0
    for _, weight in pairs(up_nodes) do
        -- the weight can be zero
        safe_limit = safe_limit + weight + 1
    end

    local picker = roundrobin:new(up_nodes)
    local nodes_count = nkeys(up_nodes)
    return {
        upstream = upstream,
        get = function (ctx)   --- get方法
            if ctx.balancer_tried_servers and ctx.balancer_tried_servers_count == nodes_count then
                return nil, "all upstream servers tried"
            end

            local server, err
            for i = 1, safe_limit do
                server, err = picker:find()
                if not server then
                    return nil, err
                end
                if ctx.balancer_tried_servers then
                    if not ctx.balancer_tried_servers[server] then
                        break
                    end
                else
                    break
                end
            end

            return server
        end,
        after_balance = function (ctx, before_retry)  --after_balance方法
            if not before_retry then
                if ctx.balancer_tried_servers then
                    core.tablepool.release("balancer_tried_servers", ctx.balancer_tried_servers)
                    ctx.balancer_tried_servers = nil
                end

                return nil
            end

            if not ctx.balancer_tried_servers then
                ctx.balancer_tried_servers = core.tablepool.fetch("balancer_tried_servers", 0, 2)
            end

            ctx.balancer_tried_servers[ctx.balancer_server] = true
            ctx.balancer_tried_servers_count = (ctx.balancer_tried_servers_count or 0) + 1
        end,
        before_retry_next_priority = function (ctx) --方法
            if ctx.balancer_tried_servers then
                core.tablepool.release("balancer_tried_servers", ctx.balancer_tried_servers)
                ctx.balancer_tried_servers = nil
            end

            ctx.balancer_tried_servers_count = 0
        end,
    }
end
```
这个负载算法主要在`apisix/balancer/roundrobin.lua`下实现的，依赖了OpenResty的[roundrobin](https://github.com/openresty/lua-resty-balancer/blob/master/lib/resty/roundrobin.lua)，
`get`方法就是前面`pick_server`函数中`picker_server.get`，这时已经选择合适的上游server。

## balancer阶段
这个阶段设置`http_balancer_phase`，这个阶段对应OpenResty的`balancer_by_lua`。这里主要将挑选的server调用OpenResty的`balancer.set_current_peer`设置当前需要转发的`server`，同时设置
超时以及重试次数等参数。

## header_filter阶段
设置函数`http_header_filter_phase`这里主要是设置响应头

## body_filter节点
设置函数`http_body_filter_phase`

## log阶段
设置函数`http_log_phase`

这就是一个HTTP请求处理流程，APISIX也是在stream，stream处理流程跟HTTP类型。

## 引用
1. [apisix源码分析](https://shoujo.ink/2021/09/apisix-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/)

2. [APISIX源码分析——路由匹配](https://juejin.cn/post/6933768239008874510)

3. [kong的事件和缓存](https://ms2008.github.io/2018/06/11/kong-events-cache/#2-worker-%E4%BA%8B%E4%BB%B6)

4. [OpenResty的balancer](https://github.com/openresty/lua-resty-balancer)
