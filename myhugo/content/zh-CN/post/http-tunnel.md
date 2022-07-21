---
title: "Http隧道"
date: 2022-07-21T14:45:52+08:00
tags:
   - http协议 
   - http隧道 
categories:
   - http协议 
   - http隧道 
toc: true
---

## HTTP隧道代理模式
在`HTTP`协议中，`CONNECT`方法可以开启一个客户端与所请求资源之间的双向沟通的通道。它可以用来创建隧道(tunnel)。

例如，`CONNECT`可以用来访问采用了`SSL (HTTPS)`协议的站点。客户端要求代理服务器将`TCP`连接作为通往目的主机隧道。
之后该服务器会代替客户端与目的主机建立连接。连接建立好之后，代理服务器会面向客户端发送或接收`TCP`消息流。

只有当浏览器配置为使用代理服务器时才会用到`CONNECT`方法。

HTTP隧道代理跟HTTP正向代理类似。

下面是一个代理的请求

```shell
# -x 执向代理服务

$ curl https://github.com -sv -x localhost:3128
* Connected to localhost (127.0.0.1) port 3128 (#0)
* allocate connect buffer!
* Establish HTTP proxy tunnel to github.com:443
> CONNECT github.com:443 HTTP/1.1
> Host: github.com:443
> User-Agent: curl/7.64.1
> Proxy-Connection: Keep-Alive
>
< HTTP/1.1 200 Connection Established            --.
< Proxy-agent: nginx                               | custom CONNECT response
< X-Proxy-Connected-Addr: 13.229.188.59:443      --'
...
```

代理请求链路：
![01](./http-tunnel.png)

## 引用
1. [HTTP CONNECT](https://blog.csdn.net/JunChow520/article/details/115719033)

2. [理解HTTP CONNECT通道](https://www.joji.me/zh-cn/blog/the-http-connect-tunnel/)
