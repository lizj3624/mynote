---
title: "云原生API网关APISIX简介"
date: 2022-03-06T21:59:54+08:00
tags:
   - apisix 
   - cloudnative
categories:
   - apisix 
   - cloudnative 
toc: true
---

## 缘起
APISIX是国内在github开源社区比较活跃的云原生API网关，目前(2022-03-06)github star 8.6k，fork 1.6k。APISIX底层是基于Nginx和OpenResty，
我本人在工作中也是经常用到Nginx和OpenResty，因此对这个项目比较感兴趣，最近开始研究它，先了解它的架构以及一些概念。

## 架构

APISIX最底层是基于Nginx，再上一次层是OpenResty，再上一次层是ngx_lua开发的APISIX core，再上一层是APISIX Plugin Runtime(插件运行时)，
最上层是APISIX的插件，它还支持多语言(go, java等)的插件。
![01](./flow-software-architecture.png)

插件化是APISIX架构设计很不错的地方，可以使APISIX易扩展，集成了大量丰富的插件，通过rpc的方式支持多语言插件。插件请求的很多阶段被调用。
![02](./flow-load-plugin.png)

## 一些概念

### [Route路由](https://apisix.apache.org/zh/docs/apisix/architecture-design/route)
`route`路由是请求进入APISIX后，根据一定匹配规则，将请求流量转发到指定upstream或者service。    
> 如果router配置upstream和service时，优先使用upstream     
路由包含三部分：
- 匹配规则(比如: `uri`, `host`, `remote_addr`)
- 插件匹配(比如: 限流插件)
- upstream上游信息

### [Plugin插件](https://apisix.apache.org/zh/docs/apisix/architecture-design/plugin)
`Plugin`是请求/响应过程中执行的插件配置。`Plugin`配置可直接绑定在`Route`上，也可以被绑定在`Service`或`Consumer`上。
而对于同一个插件的配置，只能有一份是有效的，配置选择优先级总是`Consumer > Route > Service`。
一个插件在一次请求中只会执行一次，即使被同时绑定到多个不同对象中（比如`Route`或`Service`）

### [Service服务](https://apisix.apache.org/zh/docs/apisix/architecture-design/service)
`Service`是某类服务的抽象，是有Plugin和upstream组成的一组服务，Plugin可以选的，它通常与上游`upstream`服务抽象是一一对应的。
`Route`与`Service`之间，通常是`N:1`的关系。

### [Consumer消费者](https://apisix.apache.org/zh/docs/apisix/architecture-design/consumer)
`Consumer`是某类服务的消费者，需与用户认证体系配合才能使用。 比如不同的`Consumer`请求同一个`API`，
网关服务根据当前请求用户信息，对应不同的`Plugin`或`Upstream`配置。

### [Upstream上游](https://apisix.apache.org/zh/docs/apisix/architecture-design/upstream)
`Upstream`是虚拟主机抽象，对给定的多个服务节点按照配置规则进行负载均衡。`Upstream`的地址信息可以直接配置到`Route`(或`Service`) 上，
当`Upstream`有重复时，就需要用“引用”方式避免重复了。
