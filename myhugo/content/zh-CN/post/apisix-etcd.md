---
title: "APISIX通过Etcd查找route以及service等的机制"
date: 2022-04-10T17:42:33+08:00
tags:
   - apisix
   - etcd 
categories:
   - apisix
   - etcd 
toc: true
---

apisix通过etcd作为后端存储，存储了route、service、plugin、upstream等信息，我们看一下如何通过etcd查找路由等信息，如果路由有变化时时如何通知更新的。
apisix也是支持yaml文件存储的，我们主要介绍etcd作为存储。

apisix与etcd交互是通过[resty-lua-etcd](https://github.com/api7/lua-resty-etcd)，这个也是apisix自己开发并开源的组件。apisix的etcd核心代码都在[config_etc.lua](https://github.com/apache/apisix/blob/master/apisix/core/config_etcd.lua)。

### 启动阶段start

调用`etcd.init()`根据配置初始化etcd，创建etcd的client，测试验证是否ok。

```lua
local function start(env, ...)
    ...
    init(env)
    init_etcd(env, args)   ---调用etcd.init()

    util.execute_cmd(env.openresty_args)
end

-- config_etcd.lua
function _M.init()
    local local_conf, err = config_local.local_conf()
    if not local_conf then
        return nil, err
    end

    if table.try_read_attr(local_conf, "apisix", "disable_sync_configuration_during_start") then
        return true
    end

    local etcd_cli, err = get_etcd()
    if not etcd_cli then
        return nil, "failed to start a etcd instance: " .. err
    end

    local etcd_conf = local_conf.etcd
    local prefix = etcd_conf.prefix
    local res, err = readdir(etcd_cli, prefix, create_formatter(prefix))
    if not res then
        return nil, err
    end

    return true
end
```

### init_worker阶段

在这个阶段调用各个组件模块的`init_woker`，从etcd中获取router、service、plugin、upstream等信息，我看一下

```lua
-- init.lua中http_init_worker在init_worker阶段调用
function _M.http_init_worker()
    ...
    require("apisix.balancer").init_worker()
    load_balancer = require("apisix.balancer")
    require("apisix.admin.init").init_worker()

    require("apisix.timers").init_worker()

    require("apisix.debug").init_worker()

    ...
    plugin.init_worker()
    router.http_init_worker()  --route的init_worker函数
    require("apisix.http.service").init_worker()   --- service的init_worker函数
    plugin_config.init_worker()
    require("apisix.consumer").init_worker()

    apisix_upstream.init_worker()  -- upstream的init_worker
    ...
end
```

着重看一下route的`init_worker` 函数

```lua
function _M.http_init_worker()
    local conf = core.config.local_conf()
    local router_http_name = "radixtree_uri"
    local router_ssl_name = "radixtree_sni"

    if conf and conf.apisix and conf.apisix.router then
        router_http_name = conf.apisix.router.http or router_http_name
        router_ssl_name = conf.apisix.router.ssl or router_ssl_name
    end

    -- router_http_name引用相应的路由模块apisix/http/router
    local router_http = require("apisix.http.router." .. router_http_name)
    --绑定路由模块的init_worker函数和routes函数，init_worker不存在时就apisix/http/route.lua中的init_worker
    attach_http_router_common_methods(router_http)
    --调用模块的init_worker函数，一般都是apisix/http/route.lua中的init_worker函数
    router_http.init_worker(filter)
    _M.router_http = router_http

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

我们在跟进`router_http.init_worker(filter)`这个就是调用的`apisix/http/route.lua`中的`init_worker`函数，并将从etcd获取的路由信息存储在`http_router.user_routes`中，以便`access`阶段进行路由匹配。

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

这里`core.config.new`对应是`apisix/core/config_etcd.lua`的`new`函数，因为`core.config`设置的配置中心`etcd`，这是获取route的信息因此key是`routes`，获取到信息存储到`user_routes`变量中，有变更时就更新这个变量，再次根据`etcd`的`new`函数。

```lua
function _M.new(key, opts)
    ...
    local obj = setmetatable({
        etcd_cli = nil,
        key = key and prefix .. key,
        automatic = automatic,
        item_schema = item_schema,
        checker = checker,
        sync_times = 0,
        running = true,
        conf_version = 0,    ---设置当前版本函数
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
        --- 定时函数，定时检查配置是否有变更，变更时就更新
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

etcd的这个`new`定义对象`obj`，将每次通过定时处理函数是`_automatic_fetch`从etcd中获取的放到这个对象的`values`，并更新`conf_version`的版本号，我们跟进这个定时函数

```lua
local function _automatic_fetch(premature, self)
    if premature then
        return
    end

    if not (health_check.conf and health_check.conf.shm_name) then
        local _, err = health_check.init({
            shm_name = health_check_shm_name,
            fail_timeout = self.health_check_timeout,
            max_fails = 3,
            retry = true,
        })
        if err then
            log.warn("fail to create health_check: " .. err)
        end
    end

    local i = 0
    while not exiting() and self.running and i <= 32 do
        i = i + 1

        local ok, err = xpcall(function()
            if not self.etcd_cli then
                local etcd_cli, err = get_etcd()  -- 获取etcd客户端
                if not etcd_cli then
                    error("failed to create etcd instance for key ["
                          .. self.key .. "]: " .. (err or "unknown"))
                end
                self.etcd_cli = etcd_cli
            end

            local ok, err = sync_data(self)   -- 从etcd中get数据
            if err then
                if string.find(err, err_etcd_unhealthy_all) then
                    local reconnected = false
                    while err and not reconnected and i <= 32 do
                        local backoff_duration, backoff_factor, backoff_step = 1, 2, 6
                        for _ = 1, backoff_step do
                            i = i + 1
                            ngx_sleep(backoff_duration)
                            _, err = sync_data(self)
                            if not err or not string.find(err, err_etcd_unhealthy_all) then
                                log.warn("reconnected to etcd")
                                reconnected = true
                                break
                            end
                            backoff_duration = backoff_duration * backoff_factor
                            log.error("no healthy etcd endpoint available, next retry after "
                                       .. backoff_duration .. "s")
                        end
                    end
                elseif err ~= "timeout" and err ~= "Key not found"
                    and self.last_err ~= err then
                    log.error("failed to fetch data from etcd: ", err, ", ",
                              tostring(self))
                end

                if err ~= self.last_err then
                    self.last_err = err
                    self.last_err_time = ngx_time()
                else
                    if ngx_time() - self.last_err_time >= 30 then
                        self.last_err = nil
                    end
                end

                -- etcd watch timeout is an expected error, so there is no need for resync_delay
                if err ~= "timeout" then
                    ngx_sleep(self.resync_delay + rand() * 0.5 * self.resync_delay)
                end
            elseif not ok then
                -- no error. reentry the sync with different state
                ngx_sleep(0.05)
            end

        end, debug.traceback)

        if not ok then
            log.error("failed to fetch data from etcd: ", err, ", ",
                      tostring(self))
            ngx_sleep(self.resync_delay + rand() * 0.5 * self.resync_delay)
            break
        end
    end

    if not exiting() and self.running then
        ngx_timer_at(0, _automatic_fetch, self)  --再次设置定时器
    end
end
```

`init_worker`阶段etcd最重要的功能就是定时回调函数，定时同步etcd中的数据放到`user_routes`。到这里`init_worker`介绍完了。



### access阶段

这个阶段主要匹配路由和挑选server。

```lua
function _M.http_access_phase()
    local ngx_ctx = ngx.ctx
    ...
    -- 匹配路由
    router.router_http.match(api_ctx)
    ...
    -- 挑选server
    local server, err = load_balancer.pick_server(route, api_ctx)
    if not server then
        core.log.error("failed to pick server: ", err)
        return core.response.exit(502)
    end
    ...
end
```

#### 匹配路由

apisix为了提高查找性能，基于基数树写一套路由查找的插件[GitHub - api7/lua-resty-radixtree: Adaptive Radix Trees implemented in Lua / LuaJIT](https://github.com/api7/lua-resty-radixtree)，`match`函数主要调用[apisix/radixtree_uri.lua at master · apache/apisix · GitHub](https://github.com/apache/apisix/blob/master/apisix/http/router/radixtree_uri.lua)中的`match`，这个可以根据`config_default.ymal`配置修改用那种路由查找模块，有三个路由查找`radixtree_uri.lua、radixtree_host_uri.lua、radixtree_uri_with_parameter.lua`，我们先来看一下这个`match`函数

```lua
function _M.match(api_ctx)
    local user_routes = _M.user_routes   --从etcd获取的路由数据
    local _, service_version = get_services()  --service版本
    --user_routes的数据有变更时重新创建radixtree的路由查找树
    if not cached_router_version or cached_router_version ~= user_routes.conf_version
        or not cached_service_version or cached_service_version ~= service_version
    then
        uri_router = base_router.create_radixtree_uri_router(user_routes.values,
                                                             uri_routes, false)
        cached_router_version = user_routes.conf_version
        cached_service_version = service_version
    end

    if not uri_router then
        core.log.error("failed to fetch valid `uri` router: ")
        return true
    end

    --真正路由匹配
    return base_router.match_uri(uri_router, match_opts, api_ctx)
end
``
```

`match`函数有两个重要的函数`create_radixtree_uri_router`和`base_router.match_uri`，前者主要在从etcd中route信息的`user_routes`版本有变更时，将route信息`user_routes.values`重建radixtree的路由查找树，后者是将请求匹配radixtree的路由查找树，查合适的路由信息。

#### 挑选server

挑选server主要是`load_balancer.pick_server`这个函数主要是调用的`apisix/balancer.lua`中的`pick_server`函数

```lua
local function pick_server(route, ctx)
    ...
    local server_picker = ctx.server_picker
    if not server_picker then
        -- 根据负载均衡类型选取合适的lru缓存
        server_picker = lrucache_server_picker(key, version,
                                               create_server_picker, up_conf, checker)
    end
    if not server_picker then
        return nil, "failed to fetch server picker"
    end

    local server, err = server_picker.get(ctx)  --合适balancer的get方法
    if not server then
        err = err or "no valid upstream node"
        return nil, "failed to find valid upstream server, " .. err
    end
    ctx.balancer_server = server
    ...
end
```

这里有两个重要的函数`lrucache_server_picker`和`server_picker.get`函数，前者创建balancer的lru缓存，lru的new函数是`lrucache.lua`的`new_lru_fun`函数，通过`create_obj_fun`创建缓存对象，在缓存击穿后为了保证只有一个worker进程取etcd获取数据，使用了`resty.lock`锁。

```lua
local function new_lru_fun(opts)
    local item_count, item_ttl
    if opts and opts.type == 'plugin' then
        item_count = opts.count or PLUGIN_ITEMS_COUNT
        item_ttl = opts.ttl or PLUGIN_TTL
    else
        item_count = opts and opts.count or GLOBAL_ITEMS_COUNT
        item_ttl = opts and opts.ttl or GLOBAL_TTL
    end

    local item_release = opts and opts.release
    local invalid_stale = opts and opts.invalid_stale
    local serial_creating = opts and opts.serial_creating
    local lru_obj = lru_new(item_count)

    return function (key, version, create_obj_fun, ...)
        if not serial_creating or not can_yield_phases[get_phase()] then
            local cache_obj = fetch_valid_cache(lru_obj, invalid_stale,
                                item_ttl, item_release, key, version)
            if cache_obj then
                return cache_obj.val
            end

            local obj, err = create_obj_fun(...)  --回调函数就是create_server_picker
            if obj ~= nil then
                lru_obj:set(key, {val = obj, ver = version}, item_ttl)
            end

            return obj, err
        end

        local cache_obj = fetch_valid_cache(lru_obj, invalid_stale, item_ttl,
                            item_release, key, version)
        if cache_obj then
            return cache_obj.val
        end

        local lock, err = resty_lock:new(lock_shdict_name)
        if not lock then
            return nil, "failed to create lock: " .. err
        end

        local key_s = tostring(key)
        log.info("try to lock with key ", key_s)

        local elapsed, err = lock:lock(key_s)
        if not elapsed then
            return nil, "failed to acquire the lock: " .. err
        end

        cache_obj = fetch_valid_cache(lru_obj, invalid_stale, item_ttl,
                        nil, key, version)
        if cache_obj then
            lock:unlock()
            log.info("unlock with key ", key_s)
            return cache_obj.val
        end

        local obj, err = create_obj_fun(...)
        if obj ~= nil then
            lru_obj:set(key, {val = obj, ver = version}, item_ttl)
        end
        lock:unlock()
        log.info("unlock with key ", key_s)

        return obj, err
    end
end
```

`create_obj_fun`的回调函数是`create_server_picker`

```lua
local function create_server_picker(upstream, checker)  --upstream是从etcd中获取数据，ctx.upstream_conf
    local picker = pickers[upstream.type]
    if not picker then
        -- 根据不同负载均衡算法挑选合适负载均衡模块，upstream.type是负载算法类型
        pickers[upstream.type] = require("apisix.balancer." .. upstream.type)
        picker = pickers[upstream.type]
    end

    if picker then
        local nodes = upstream.nodes
        local addr_to_domain = {}
        for _, node in ipairs(nodes) do
            if node.domain then
                local addr = node.host .. ":" .. node.port
                addr_to_domain[addr] = node.domain
            end
        end

        local up_nodes = fetch_health_nodes(upstream, checker)

        if #up_nodes._priority_index > 1 then
            core.log.info("upstream nodes: ", core.json.delay_encode(up_nodes))
            local server_picker = priority_balancer.new(up_nodes, upstream, picker)
            server_picker.addr_to_domain = addr_to_domain
            return server_picker
        end

        core.log.info("upstream nodes: ",
                      core.json.delay_encode(up_nodes[up_nodes._priority_index[1]]))
        -- 调用balancer的new方法
        local server_picker = picker.new(up_nodes[up_nodes._priority_index[1]], upstream)
        server_picker.addr_to_domain = addr_to_domain
        return server_picker
    end

    return nil, "invalid balancer type: " .. upstream.type, 0
end
```

`pickers`变量存放负载均衡算法的模块，apisix支持的负载算法`roundrobin、chash、least_conn`，这些实现代码模块都在`apisix/balancer`目录，`picker.new`是balancer对象的`new`方法。我们看一下`roundrobin`的`balancer`

```lua
function _M.new(up_nodes, upstream)  --new方法
    local safe_limit = 0
    for _, weight in pairs(up_nodes) do
        -- the weight can be zero
        safe_limit = safe_limit + weight + 1
    end

    local picker = roundrobin:new(up_nodes)
    local nodes_count = nkeys(up_nodes)
    return {
        upstream = upstream,
        get = function (ctx)   --get方法
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
        before_retry_next_priority = function (ctx)  -- before_retry_next_priority方法
            if ctx.balancer_tried_servers then
                core.tablepool.release("balancer_tried_servers", ctx.balancer_tried_servers)
                ctx.balancer_tried_servers = nil
            end

            ctx.balancer_tried_servers_count = 0
        end,
    }
end
```

前面的`server_picker.get`就是调用这里的`get`方法。apisix中的upstream server放到lru的缓存中，所有的worker共享这个缓存，缓存击穿后通过`resty.lock`保证只有一个worker去etcd中获取数据。

到这里apisix从etcd获取数据，有变更时更新缓存数据的流程介绍完了。
