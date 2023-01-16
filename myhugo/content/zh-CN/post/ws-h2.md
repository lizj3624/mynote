---
title: "WebSocket与HTTP2对比"
date: 2023-01-16T20:36:10+08:00
tags:
   - HTTP 
   - WebSocket
   - 协议
categories:
   - HTTP 
   - WebSocket
   - 协议
toc: true
---

WebSocket协议是基于TCP的一种新的网络协议。它实现了浏览器与服务器全双工（full-duplex）通信，即允许服务器主动发送信息给客户端。因此，在WebSocket中，浏览器和服务器只需要完成一次握手，两者之间就直接可以创建持久性的连接，并进行双向数据传输，客户端和服务器之间的数据交换变得更加简单。

HTTP2协议是基于二进制分帧来高效传输数据的HTTP协议，支持多路复用解决了线头阻塞(HOL Blocking)，通过HPack的头部压缩技术降低负载，也支持服务器推送让服务器能够主动推送响应到客户端缓存中。

这两个协议都支持服务端推送，HTTP2可以替代WebSocket吗？解决这个疑问，得先分析这两个协议。

## WebSocket

websocket是一个双向通信协议，它在握手阶段采用http1.1，建立TCP后开始传输WebSocket格式通信数据。

### 握手过程

发起握手请求

```http
HTTP/1.1 101 Switching Protocols  // 状态行
Upgrade: websocket   // required
Connection: Upgrade  // required
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo= // required，加密后的 Sec-WebSocket-Key
Sec-WebSocket-Protocol: chat // 表明选择的子协议
```

服务器如果支持websocket，返回101响应

```http
HTTP/1.1 101 Switching Protocols  // 状态行
Upgrade: websocket   // required
Connection: Upgrade  // required
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo= // required，加密后的 Sec-WebSocket-Key
Sec-WebSocket-Protocol: chat // 表明选择的子协议
```

WebSocket提供两种协议：不加密的ws:// 和 加密的 wss://。
因为是用 HTTP 握手，它和 HTTP 使用同样的端口：ws是80（HTTP），wss是 43（HTTPS）

### WebSocket通信格式

![websocket](./websocket.png)

1. FIN: 1位，用于描述消息是否结束，如果为1则该消息为消息尾部,如果为零则还有后续数据包。

2. RSV1,RSV2,RSV3：各1位，用于扩展定义的,如果没有扩展约定的情况则必须为0。

3. OPCODE: 4位，用于表示消息接收类型，如果接收到未知的opcode，接收端必须关闭连接。长连接探活包就是这里标识的。
   　　OPCODE定义的范围：
   
       0x0表示附加数据帧
   
   　　　　0x1表示文本数据帧
   　　　　0x2表示二进制数据帧
   　　　　0x3-7暂时无定义，为以后的非控制帧保留
   　　　　0x8表示连接关闭
   　　　　0x9表示ping
   　　　　0xA表示pong
   　　　　0xB-F暂时无定义，为以后的控制帧保留

4. MASK: 1位，用于标识PayloadData是否经过掩码处理，客户端发出的数据帧需要进行掩码处理，所以此位是1。数据需要解码。

5. Payload length

　　如果 x值在0-125，则是payload的真实长度。

　　如果 x值是126，则后面2个字节形成的16位无符号整型数的值是payload的真实长度。

　　如果 x值是127，则后面8个字节形成的64位无符号整型数的值是payload的真实长度。

　　此外，如果payload length占用了多个字节的话，payload length的二进制表达采用网络序（big endian，重要的位在前）。

### 使用场景

1. 弹幕：终端用户A在自己的手机端发送了一条弹幕信息，但是您也需要在客户A的手机端上将其他N个客户端发送的弹幕信息一并展示。需要通过WebSocket协议将其他客户端发送的弹幕信息从服务端全部推送至客户A的手机端，从而使客户A可以同时看到自己发送的弹幕和其他用户发送的弹幕。

2. 在线教育：老师进行一对多的在线授课，在客户端内编写的笔记、大纲等信息，需要实时推送至多个学生的客户端，需要通过WebSocket协议来完成。

3. 股票等金融产品实时报价股：股票黄金等价格变化迅速，变化后，可以通过WebSocket协议将变化后的价格实时推送至世界各地的客户端，方便交易员迅速做出交易判断。

4. 体育实况更新：由于全世界体育爱好者数量众多，因此比赛实况成为其最为关心的热点。这类新闻中最好的体验就是利用WebSocket达到实时的更新。

5. 视频会议和聊天：尽管视频会议并不能代替和真人相见，但是应用场景众多。WebSocket可以帮助两端或多端接入会议的用户实时传递信息。

6. 基于位置的应用：越来越多的开发者借用移动设备的GPS功能来实现基于位置的网络应用。如果您一直记录终端用户的位置（例如：您的 App记录用户的运动轨迹），就可以收集到更加细致化的数据。

## HTTP/2

![http2](./http2.png)

1. 二进制分帧
   http2.0之所以能够突破http1.X标准的性能限制，改进传输性能，实现低延迟和高吞吐量，就是因为其新增了二进制分帧层。

帧(frame)包含部分：类型Type, 长度Length, 标记Flags, 流标识Stream和frame payload有效载荷。

消息(message)：一个完整的请求或者响应，比如请求、响应等，由一个或多个 Frame 组成。

流是连接中的一个虚拟信道，可以承载双向消息传输。每个流有唯一整数标识符。为了防止两端流ID冲突，客户端发起的流具有奇数ID，服务器端发起的流具有偶数ID。

2. 多路复用
   多路复用允许同时通过单一的HTTP/2连接发起多重的请求-响应消息。有了新的分帧机制后，HTTP/2不再依赖多个TCP连接去实现多流并行了。

3. 头部压缩
   HTTP/2使用encoder来减少需要传输的header大小，通讯双方各自缓存一份头部字段表，既避免了重复header的传输，又减小了需要传输的大小。

4. 服务端推送
   ![sp](./serverpush.png)
   服务器可以对一个客户端请求发送多个响应，服务器向客户端推送资源无需客户端明确地请求。并且服务端推送能把客户端所需要的资源伴随着index.html一起发送到客户端，省去了客户端重复请求的步骤。

注意两点：
1、推送遵循同源策略；

2、这种服务端的推送是基于客户端的请求响应来确定的。

## 对比

#### HTTP/2与WebSocket比较

|         | HTTP/2                                                            | WebSocket         |
| ------- | ----------------------------------------------------------------- | ----------------- |
| Headers | Compressed (HPACK) 请求头部压缩                                         | None              |
| Binary  | Yes                                                               | Binary or Textual |
| 多路复用    | Yes                                                               | Yes               |
| 优先级     | Yes                                                               | No                |
| 压缩      | Yes                                                               | Yes               |
| 传送方向    | Client/Server + Server Push (Server Push只能浏览器消化，不支持API,也就是代码无法使用) | 双向                |
| 全双工     | Yes                                                               | Yes               |

HTTP/2中服务端推送跟WebSocket的区别

- HTTP2 Server Push，一般用以服务器根据解析 `index.html` 同时推送 `JPG/JS/CSS` 等资源，而免了服务器发送多次请求。
- websocket，用以服务器与客户端手动编写代码去推送进行数据通信。

HTTP/2不是类似于Websocket或者SSE这样的推送技术的替代品。将HTTP/2和SSE（Server-Sent Events SSE）结合起来提供高效的基于HTTP的双向通信。Websocket技术可能会继续使用，但是SSE和其EventSource API同HTTP/2的能力相结合可以在多数场景下达到同样的效果，但是会更简单。

> SSE(Server-Sent Events)服务器发送事件，SSE是一种让服务器能够在客户端服务器建立连接之后异步推送数据给客户端的机制。这样服务器就可以在数据“块”准备好之后再发送数据。这可以被看成是一种单向的发布订阅模型。多数的现代浏览器也实现了W3C HTML5标准中一个叫做EventSource的标准JavaScript客户端API。那些不支持EventSource API的浏览器，也可以方便地通过填充的方式来获得相应的支持。

## 引用

1. [rfc6455 WebSocket协议](https://www.rfc-editor.org/rfc/rfc6455)
2. [有了HTTP/2，Websocket还有市场吗？](https://blog.csdn.net/cnweike/article/details/116056371)
3. [HTTP2和WebSocket](https://www.cnblogs.com/yrxing/p/15799678.html)
4. [WebSocket的诞生](https://www.cnblogs.com/zhangmingda/p/12678630.html)
