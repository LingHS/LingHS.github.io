---
title: http2
date: 2020-07-19 10:07:52
tags: HTTP
categories:
---

# HTTP2

不知不觉，很多网站已经开始使用 http2 的协议，甚至 google 已经开始使用特定的 http3 了。
相比于 http1.1，http2 的核心 **二进制分帧**
其他的：多路复用，压缩头部，服务器推送，请求优先级

在应用层与传输层之间增加一个二进制分帧层，以此达到“在不改动 HTTP 的语义，HTTP 方法、状态码、URI 及首部字段的情况下，突破 HTTP1.1 的性能限制，改进传输性能，实现低延迟和高吞吐量。”

在二进制分帧层上，HTTP2.0 会将所有传输的信息分割为更小的消息和帧,并对它们采用二进制格式的编码，其中 HTTP1.x 的首部信息会被封装到 Headers 帧，而我们的 request body 则封装到 Data 帧里面。

<!--- more --->

客户端和服务器可以把 HTTP 消息分解为互不依赖的帧，然后乱序发送，最后再在另一端把它们重新组合起来。注意，同一链接上有多个不同方向的数据流在传输。客户端可以一边乱序发送 stream，也可以一边接收者服务器的响应，而服务器那端同理。

原先只能一个一个按顺序发送资源，而现在则没有这个限制

**参考：**

https://www.cnblogs.com/nuannuan7362/p/10397536.html

<iframe src="//player.bilibili.com/player.html?aid=455561284&bvid=BV1p541147LD&cid=189611998&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
