---
title: "APISIX的Plugin介绍"
date: 2022-03-18T16:48:16+08:00
tags:
   - apisix
   - plugin
   - cloudnative
categories:
   - apisix
   - cloudnative
toc: true
---

## 插件机制
APISIX通过插件(Plugin)机制来丰富其功能，目前通过lua实现了8大类插件，还通过`sidecar`模式支持多语言插件。

Plugin配置可直接绑定在`Route`上，也可以被绑定在`Service`或`Consumer上`。
而对于同一个插件的配置，只能有一份是有效的，配置选择优先级总是`Consumer > Route > Service`。

一个插件在一次请求中只会执行一次，即使被同时绑定到多个不同对象中（比如`Route`或`Service`）。 
插件运行先后顺序是根据插件自身的优先级(`priority`)来决定的。

## 插件热加载
APISIX的插件是热加载的，不管你是新增、删除还是修改插件，都不需要重启服务。

只需要通过`admin API`发送一个`HTTP`请求即可：
```shell
curl http://127.0.0.1:9080/apisix/admin/plugins/reload -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT
```

## 路由加入插件
路由(route)接口中加入`plugins`关键字后支持插件，`limit-count`、`prometheus`都是引用的插件名称。

```shell
curl "http://127.0.0.1:9080/apisix/admin/routes" -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" -X POST -d '{
    "methods": ["GET"],
    "host": "test.my.com",
    "uri": "/mypath03/plugin",
    "plugins": {
        "limit-count": {
            "count": 2,
            "time_window": 60,
            "rejected_code": 503,
            "key": "remote_addr"
        },
        "prometheus": {}
    },
    "upstream_id": "00000000000000000122"
}'
```

## 多语言的外部插件
APISIX通过`Sidecar`的方式加载和运行多语言开发的插件。这里的`Sidecar`就是`Plugin Runner`，多语言开发的插件叫做`External Plugin`。
如果APISIX中配置了一个`Plugin Runner`，APISIX将以子进程的方式运行该`Plugin Runner`。 
该子进程与APISIX进程从属相同用户。当重启或者重新加载APISIX时，该`Plugin Runner`也将被重启。
请求将触发从APISIX到`Plugin Runner`的`RPC`调用。
目前支持`go、python、java、javascript`语言开发插件。
![01](./external-plugin.png)

## Lua插件开发
如果社区的插件不能满足需求，APISIX也支持自助开发插件。[Lua插件规范](https://apisix.apache.org/zh/docs/apisix/plugin-develop)

## 引用
1. [apisix plugin](https://apisix.apache.org/zh/docs/apisix/architecture-design/plugin)

2. [apisix plugin列表](https://apisix.apache.org/zh/docs/apisix/plugins/batch-requests/)

3. [apisix 外部插件](https://apisix.apache.org/zh/docs/apisix/external-plugin)

4. [apisix lua插件开发](https://apisix.apache.org/zh/docs/apisix/plugin-develop)

5. [apisix admin API](https://apisix.apache.org/zh/docs/apisix/admin-api)
