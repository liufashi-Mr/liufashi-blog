---
title: 从 HTTP 1.0 到 HTTP 3.0
date: 2022-09-28 14:28:50
categories:
  - 计算机网络
tags:
  - HTTP
---

## 从 HTTP 1.0 到 HTTP 1.1

1. 连接方面，http1.0 默认使用非持久连接，需要设置 Connection: keep-alive 开启持久连接， 而 http1.1 使用持久连接，http1.1 通过持久连接来使多个 http 请求复用同一个 tcp 连接，一次来避免 http1.0 非持久连接每次需要建立连接的时延
2. 资源请求方面，http1.0 中存在一些浪费带宽的现象，http1.1 中引入了 range 头域，允许之请求资源的某个部分，返回 206 的状态码表示进行了范围请求，这样方便开发者自由的选择和充分利用带宽和连接
3. 缓存方面，http 的缓存判断是通过 header 里的 Expires 和 If-Modify-Since 作为判断标准，http1.1 中引入了更多的策略，如 Etag、If-Unmodified-Since、If-Match、If-None-Match 等更多可选择的缓存头来控制
4. 新增方面，http1.1 新增了 host 字段，用来指定服务器的域名，随着虚拟主机的出现一台物理服务器行可以存在多个虚拟主机，他们共用一个 ip，http1.0 中认为每台服务器只能绑定一个唯一的 ip 就不成立了，因此添加了 host 字段，这样就可以将请求发往同一服务器上的不同网站，http1.1 还新增了 put head options 等请求方法

## HTTP 1.1 到 HTTP 2.0

1. 二进制协议，http2.0 是一个二进制的协议，在 http1.1 中，报文的信息必须是文本（ascll 编码），数据体可以是文本也可以是二进制。到了 http2.0 则是一个测地的二进制协议，头信息和数据体都是二进制，统称为帧，分为头信息帧和数据帧，这个概念是实现下面多路复用的基础
2. 多路复用，http2.0 实现了多路复用，依然是复用 tcp 连接，但是在一个连接里客户端和服务端都可以同时发送多个请求或回应，而且不用按照顺序一一发送，这样避免了队头阻塞的问题
   队头阻塞：队头阻塞是由 HTTP 基本的“请求 - 应答”模型所导致的。HTTP 规定报文必须是“一发一收”，这就形成了一个先进先出的“串行”队列。队列里的请求是没有优先级的，只有入队的先后顺序，排在最前面的请求会被最优先处理。如果队首的请求因为处理的太慢耽误了时间，那么队列里后面的所有请求也不得不跟着一起等待，结果就是其他的请求承担了不应有的时间成本，造成了队头堵塞的现象。
3. 数据流，http2.0 使用了数据流的概念，因为 http2.0 的数据包不是按照顺序发送的，同一个连接里连续的数据包可能属于不同请求，因此需要对数据包做标记，指出他属于哪个请求的。http2.0 将每个请求或者响应的所有数据包成为一个数据流，每个数据流有唯一的编号，数据包发送是都必须标记数据流 ID，用来区分是哪个数据流
4. 头信息压缩， HTTP/2 实现了头信息压缩，由于 HTTP 1.1 协议不带状态，每次请求都必须附上所有信息。所以，请求的很多字段都是重复的，比如 Cookie 和 User Agent ，一模一样的内容，每次请求都必须附带，这会浪费很多带宽，也影响速度。HTTP/2 对这一点做了优化，引入了头信息压缩机制。一方面，头信息使用 gzip 或 compress 压缩后再发送；另一方面，客户端和服务器同时维护一张头信息表，所有字段都会存入这个表，生成一个索引号，以后就不发送同样字段了，只发送索引号，这样就能提高速度了。
5. 服务器推送： HTTP/2 允许服务器未经请求，主动向客户端发送资源，这叫做服务器推送。使用服务器推送提前给客户端推送必要的资源，这样就可以相对减少一些延迟时间。这里需要注意的是 http2 下服务器主动推送的是静态资源，和 WebSocket 以及使用 SSE 等方式向客户端发送即时数据的推送是不同的。

## HTTP 3.0

HTTP/3 基于 UDP 协议实现了类似于 TCP 的多路复用数据流、传输可靠性等功能，这套功能被称为 QUIC 协议。
