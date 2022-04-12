---
title: "APISIX的控制面"
date: 2022-04-12T08:46:36+08:00
tags:
   - apisix
   - nginx
categories:
   - apisix
   - nginx
toc: true
---

apisix的控制面主要是提供一套RESTful API接口对存储在etcd中的`route、service、plugin、upstream`等*CURD*操作。apisix有两套控制面：

1. apisix通过`lua`开发一套控制面API接口，这套代码主要在`apisix/admin`目录下。

2. 另一个是通过`golang`的API接口，还包含一套`javascript`写的dashboard，这个项目也开源了，[GitHub - apache/apisix-dashboard: Dashboard for Apache APISIX](https://github.com/apache/apisix-dashboard)。

我们主要介绍一下`apisix/admin`这个控制面接口，对应的`url path`主要是`/apisix/admin/*`，我们看一个新增route的请求，熟悉一下API接口

```shell
curl "http://127.0.0.1:9080/apisix/admin/routes" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '{
    "methods": ["GET"],
    "host": "test.my.com",
    "uri": "/",
    "upstream_id": "00000000000000000122"
}'
```

### init_worker

在`http_init_worker`函数中也会调用`admin`的`init_worker`函数`require("apisix.admin.init").init_worker()`，做一些初始化的工作。



初始化控制面接口的route信息

```lua
local uri_route = {
    {
        paths = [[/apisix/admin/*]],
        methods = {"GET", "PUT", "POST", "DELETE", "PATCH"},
        handler = run,   -- 控制面接口的回调函数
    },
    {
        paths = [[/apisix/admin/plugins/list]],
        methods = {"GET"},
        handler = get_plugins_list,  -- 查询插件的回调函数
    },
    {
        paths = reload_event,
        methods = {"PUT"},
        handler = post_reload_plugins,  -- 插件reload的回调函数
    },
}
```

### 请求处理

我们主要看一下控制面接口的回调函数`run`

```lua
local resources = {     --- 注册了处理route、service、plugin、upstream等的模块信息
    routes          = require("apisix.admin.routes"),
    services        = require("apisix.admin.services"),
    upstreams       = require("apisix.admin.upstreams"),
    consumers       = require("apisix.admin.consumers"),
    schema          = require("apisix.admin.schema"),
    ssl             = require("apisix.admin.ssl"),
    plugins         = require("apisix.admin.plugins"),
    proto           = require("apisix.admin.proto"),
    global_rules    = require("apisix.admin.global_rules"),
    stream_routes   = require("apisix.admin.stream_routes"),
    plugin_metadata = require("apisix.admin.plugin_metadata"),
    plugin_configs  = require("apisix.admin.plugin_config"),
}

local function run()
    local api_ctx = {}
    core.ctx.set_vars_meta(api_ctx)
    ngx.ctx.api_ctx = api_ctx

    -- token校验
    local ok, err = check_token(api_ctx)
    if not ok then
        core.log.warn("failed to check token: ", err)
        core.response.exit(401, {error_msg = "failed to check token"})
    end

    local uri_segs = core.utils.split_uri(ngx.var.uri)
    core.log.info("uri: ", core.json.delay_encode(uri_segs))

    -- /apisix/admin/schema/route
    local seg_res, seg_id = uri_segs[4], uri_segs[5]
    local seg_sub_path = core.table.concat(uri_segs, "/", 6)
    if seg_res == "schema" and seg_id == "plugins" then
        -- /apisix/admin/schema/plugins/limit-count
        seg_res, seg_id = uri_segs[5], uri_segs[6]
        seg_sub_path = core.table.concat(uri_segs, "/", 7)
    end

    if seg_res == "stream_routes" then
        local local_conf = core.config.local_conf()
        if not local_conf.apisix.stream_proxy then
            core.log.warn("stream mode is disabled, can not add any stream ",
                          "routes")
            core.response.exit(400, {error_msg = "stream mode is disabled, " ..
                               "can not add stream routes"})
        end
    end

    -- resources是注册了处理route、service、plugin、upstream等的模块信息
    local resource = resources[seg_res]
    if not resource then
        core.response.exit(404, {error_msg = "not found"})
    end

    local method = str_lower(get_method())
    if not resource[method] then
        core.response.exit(404, {error_msg = "not found"})
    end

    local req_body, err = core.request.get_body(MAX_REQ_BODY)
    if err then
        core.log.error("failed to read request body: ", err)
        core.response.exit(400, {error_msg = "invalid request body: " .. err})
    end

    if req_body then
        local data, err = core.json.decode(req_body)
        if err then
            core.log.error("invalid request body: ", req_body, " err: ", err)
            core.response.exit(400, {error_msg = "invalid request body: " .. err,
                                     req_body = req_body})
        end

        req_body = data
    end

    local uri_args = ngx.req.get_uri_args() or {}
    if uri_args.ttl then
        if not tonumber(uri_args.ttl) then
            core.response.exit(400, {error_msg = "invalid argument ttl: "
                                                 .. "should be a number"})
        end
    end

    -- 根据PUT/POST/PATHCE，调用相应的处理函数
    local code, data = resource[method](seg_id, req_body, seg_sub_path,
                                        uri_args)
    if code then
        data = strip_etcd_resp(data)
        core.response.exit(code, data)
    end
end
```

比如我们前面介绍的新增route的请求`curl "http://127.0.0.1:9080/apisix/admin/routes" -X POST -d '{json data}'`，根据url中`route`可以从查找`resources`到引用`require("apisix.admin.routes")`模块，根据method方法`POST(resource[method])`可以调用`routes`模块的`post`处理函数，我们就以`route`为例，跟进看一下`route`模块的路由信息的操作。

控制面对路由信息的操作主要在在`apisix/admin/routes.lua`中

```lua
-- 更新数据的校验
local function check_conf(id, conf, need_id)
    ...
end

-- route的put函数
function _M.put(id, conf, sub_path, args)
end

-- route的get函数
function _M.get(id)
end

-- route的post函数
function _M.post(id, conf, sub_path, args)
    local id, err = check_conf(id, conf, false)  --检查更新请求数据是否合法
    if not id then
        return 400, err
    end

    local key = "/routes"  -- 存储到etcd的key
    utils.inject_timestamp(conf)

    -- 调用etcd的push方法，将更新数据存储到etcd中
    local res, err = core.etcd.push(key, conf, args.ttl)
    if not res then
        core.log.error("failed to post route[", key, "] to etcd: ", err)
        return 503, {error_msg = err}
    end

    return res.status, res.body
end

-- route的delete函数
function _M.delete(id)
end

-- route的patch函数
function _M.patch(id, conf, sub_path, args)
end
```

`routes`模块实现了路由信息的`PUT/GET/POST/DELETE/PATCH`方法，这些方法都是对etcd的操作，etcd的实现操作放在`apisix/core/etcd.lua`中。其他`service/plugin/upstream`的实现跟着比较类似，这里就不在详解了。

apisix的控制面主要是通过REST API接口对存储在etcd中的`route、service、plugin、upstream`等*CURD*操作。
