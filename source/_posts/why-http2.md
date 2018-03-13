---
title: Why HTTP/2 ?
date: 2016-10-17 16:23:55
tags: 
  - HTTP2
  - HTTPS
category: 写码 
---

HTTP、HTTPS、HTTP/2的一些比较 (HTTP/2 虽然从协议角度来说并不一定要基于 TLS 但是浏览器的实现全部是基于 TLS 的，以下所说的 HTTP/2 指浏览器实现的基于 TLS 的版本)。

<!--more-->
### SEO
 | HTTP | HTTPS & HTTP2
----|------|----
谷歌|  ❌ | ✔️
百度| ❌  | ❌ 
HTTPS 对谷歌的 SEO 有微弱加成，百度后续也会有加成

### 安全
 | HTTP | HTTPS & HTTP/2
----|------|----
敏感信息泄露| ❌  | ✔️
DNS 污染| ❌  | ✔️
DNS 劫持| ❌ | ✔️
中间人攻击 | ❌  | ✔️ (连接建立后)
HTTPS 和 HTTP/2 的安全性明显优于 HTTP


### 计算耗时
相比于HTTP，HTTPS & HTTP/2 加解密过程中存在额外的计算耗时，特别是首次连接建立的 TLS 握手过程中的非对称加解密 (30ms+)。

### 首次请求建立前的 RTT
#### HTTP
 ![http][http]
 最坏耗时 2 个 RTT：
 
  * 域名解析，耗时 1 个 RTT (缓存)
  * TCP 三次握手，耗时 1 个 RTT
  
一般耗时 1 个 RTT

#### HTTPS
 ![https][https]
 最坏耗时 9 个 RTT：
 
 * 域名解析， 耗时 1 个 RTT
 * TCP 三次握手，耗时 1 个 RTT
 * HTTP 重定向 HTTPS 及后续握手 耗时 2 个 RTT  (开启 [HSTS](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security))
 		* 302 过程，耗时 1 个 RTT
 		* TCP 重新握手，耗时 1 个 RTT
 * TLS 完全握手过程 耗时 4 个 RTT + 1 个 DNS 解析 
		* TLS 完全握手 耗时 2 个 RTT
 		* TLS 握手过程中的 CA 域名解析 耗时 1 个 RTT (缓存)
 		* CA 站点 TCP 握手，耗时 1 个 RTT
 		* [OCSP](https://en.wikipedia.org/wiki/	Online_Certificate_Status_Protocol) 查询 耗时 1 个 RTT (缓存)
 		
一般耗时 2 个 RTT
 
#### HTTP/2
![ALPN][ALPN] 	
HTTP/2 使用 [ALPN](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation) 在 TLS 握手过程中即可进行, 耗时同 HTTPS


### 多路复用
	
![compare][compare] 

HTTP 和 HTTPS 在使用长连接的情况下后续请求延迟相差不大，两者都存在 HOL blocking 问题，每次请求都需要等上一个请求完成，这个过程会造成额外的 RTT。HTTP/1.1 虽然可采用

* 内嵌及打包资源以减少HTTP请求
* 多连接 (浏览器一般默认每来源 6 个连接，可利用多域名优化突破连接数限制) 减少排队 
* HTTP pipelining 

等优化手段减少 RTT 延迟，但每个方案都不是完美的，如内嵌或打包资源无法使用细粒度的缓存，多连接的方式会对服务端造成相当大的开销，HTTP pipelining 仍然存在 HOL blocking 的问题，且只对幂等的请求有效。

HTTP/2 使用二进制分帧机制，所有通信在一个 TCP 连接上完成，可以并行交错地发送请求和响应，解决了应用层 HOL 阻塞问题 (TCP 层仍存在 HOL 阻塞)，消除了并行处理和发送请求及响应时对多个连接的依赖，而且其在协议层面的支持，不需要额外的开发。
	
### 首部压缩
HTTP 的每一次通信都会携带一组首部，用于描述传输的资源及其属性。在 HTTP/1.1 中，这些元数据都是以纯文本形式发送的，重复且过大的首部增加了延迟和带宽开销，严重的情况下，由于 TCP 慢启动的原因，首部长度超过初始拥塞窗口，会增加一次额外的 RTT。

HTTP/2 采用 HPACK 压缩首部，有效的减少了重复首部字节的流量开销。

### 服务器推送
HTTP/1.1 不支持服务器推送。

HTTP/2 支持服务器推送，服务器可以对一个客户端请求发送多个响应，如此服务器可以分析资源文档，将文档相关连的资源提前推送给客户端，避免了客户端文档分析依赖后，重新请求的开销。

客户端可以拒绝服务端推送的资源，这可能会造成资源的浪费。目前大部分服务器还不支持服务器推送。

### 请求优先级
HTTP/1.1 不支持请求优先级。

HTTP/2 可以设置请求的优先级，通过优先级设置可以将重要的资源优先发送给客户端，客户端因此可以更快的向用户呈现界面。

目前大部分服务器还不支持服务器推送

### 流量控制
HTTP/1.1 不支持流量控制。

HTTP/2 采用类似 TCP 滑动窗口的机制进行流量控制，可以避免网络拥堵。

目前大部分服务器还不支持服务器推送

### 总结
* 绝绝绝大部分情况： HTTP/2 优于 HTTPS 
* 请求数量多的情况：HTTP/2 优于 HTTP
* 对安全性要求高的情况：HTTPS 优于 HTTP
* 高 RTT 的情况： HTTP 优于 HTTP/2 
* 下载大文件的情况：HTTP 优于 HTTP/2 

总之如果之前是 HTTP，且要考虑浏览器兼容，对安全传输要求不高，采用过打包，多域名等优化手段的情况，升级 HTTP/2 可能会造成负面的性能问题。除此之外，就愉快的升级 HTTP/2 吧 (特别是主要升级成本在运维大哥的情况下)。


[http]: http://ogovd1xl2.bkt.clouddn.com/http.png
[https]: http://ogovd1xl2.bkt.clouddn.com/https.png
[ALPN]: http://ogovd1xl2.bkt.clouddn.com/how-alpn-works.png
[os]: http://ogovd1xl2.bkt.clouddn.com/wwww.png
[compare]: http://ogovd1xl2.bkt.clouddn.com/123.png
