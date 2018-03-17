---
title: HTTP 2.0 与 NPN 与 ALPN 与 OpenSSL 与 Nginx
date: 2016-10-16 14:40:31
tags: 
  - HTTP2
  - NPN
  - ALPN
  - Nginx
category: 写码 
---

Chrome build 51 移除了对 NPN 和 SPDY 的支持，这导致大部分基于 NPN 的 HTTP 2.0 服务回落到 HTTP 1.1 版本。在新版及以后的Chrome中只有支持了 ALPN 的 HTTP 2.0 服务才是可用的。

<!--more-->
### NPN & ALPN
首先，还是先简单介绍下 NPN 和 ALPN

* NPN (Next Protocol Negotiation)
![NPN][NPN] 
NPN 是 Google 在 SPDY 中开发的一个 TLS 扩展。如上图所示，NPN 是利用 TLS 握手中的 ServerHello 消息，在其中追加 ProtocolNameList 字段包含自己支持的应用层协议，客户端检查改字段，并在之后的 ClientKeyExChange (图中的 Certificate Verify) 消息中以 ProtocolName 字段 返回选中的协议。

* ALPN (Application-Layer Protocol Negotiation)
![ALPN][ALPN]
ALPN 是 IETF 在 NPN 的基础上修订的版本，也是HTTP2的基准。在 ALNP 中，交换次序相对于 NPN 是颠倒的，客户端先声明自己支持的协议 (ClientHello)，服务器选择并确认协议 (ServerHello) 。这样颠倒的目的主要是使 ALPN 与其他协议协商标准保持一致 (如 SSL) 。

服务器和客户端可以在 TLS 握手过程中选择双方所支持的应用层协议进行后续的加密通信，如果不支持任何一种协议则断开通信连接。

### OpenSSL
一般的 Web 服务 (如 Nginx) 都是使用的 OpenSSL，而 OpenSSL 对 ALPN 的支持是在 2015年1月 发布的 1.0.2 版本中加入的。目前主流的操作系统的 OpenSSL 版本情况如下表所示：
![os][os]
想要启用 ALPN 则必需升级版本到 1.0.2 以上，升级命令如下：

```
$ wget https://www.openssl.org/source/openssl-1.0.2j.tar.gz
$ tar -zxf openssl-1.0.2j.tar.gz -C /usr/local/
$ cd /usr/local/openssl-1.0.2j
$ ./config
$ make
$ make test
$ make install
$ mv /usr/bin/openssl /usr/bin/openssl_1.0.1e
$ ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl
$ openssl version
OpenSSL 1.0.2j  26 Sep 2016
```
### Nginx
Nginx 是在 1.9 版本中加入对 HTTP 2.0 的支持的，在 Chrome build 51 后如需要 支持 HTTP 2.0 服务的话，需要重新构建一遍 Nginx

安装 PCRE

```
$ wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.39.tar.gz
$ tar -zxf pcre-8.39.tar.gz
$ cd pcre-8.39
$ ./configure
$ make
$ make install
```
安装 zlib

```
$ wget http://zlib.net/zlib-1.2.8.tar.gz
$ tar -zxf zlib-1.2.8.tar.gz
$ cd zlib-1.2.8
$ ./configure
$ make
$ make install
```
安装 OpenSSL 同上，安装Nginx

```
$ wget http://nginx.org/download/nginx-1.11.6.tar.gz
$ tar zxf nginx-1.11.6.tar.gz
$ cd nginx-1.11.6
$ ./configure \
    --sbin-path=/usr/local/nginx/nginx \
    --conf-path=/usr/local/nginx/nginx.conf \
    --pid-path=/usr/local/nginx/nginx.pid \
    --lock-path=/var/lock/nginx.lock \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-stream \
    --with-pcre=../pcre-8.39 \
    --with-zlib=../zlib-1.2.8 \
    --with-openssl=../openssl-1.0.2j
$ make
$ make install
$ sudo ln -s /usr/local/nginx/nginx /usr/bin
$ nginx version: nginx/1.11.6
```
之后就可以通过简单的配置开启 HTTP 2.0 服务了

```
server {
    listen 443 ssl http2;
	
	 # 加解密使用的证书和私钥
    ssl_certificate server.crt;
    ssl_certificate_key server.key;
    
    # 分割的最大响应块大小，过小会加重服务器负荷，过大会造成 HOL 阻塞
    # http2_chunk_size 8k; 
    
    # 每个请求的最大缓冲区大小 (请求被处理之前保存在缓冲区)
    # http2_body_preread_size;
    
    # 不活动连接的最大维持时间，超时后关闭连接
    # http2_idle_timeout 3m;
    
    # 同一个连接中传输的最大 HTTP 2.0 流的最大数量
    # http2_max_concurrent_streams 128;
    
    # HPACK 压缩后的最大请求头大小
    # http2_max_field_size
    
    # HPACK 解压后的最大请求头大小
    # http2_max_header_size 16k;
    
    # 单个 HTTP 2.0 连接的最大请求次数 (超过次数后启用一个新的连接) 
    # 需要 1.11.6 以上版本
    # http2_max_requests 1000;
    
    # 每个 worker 的最大接收缓冲大小
    # http2_recv_buffer_size 256k;
    
    # 期望从客户端收到更多数据的最大时间，超时后关闭连接
    # http2_recv_timeout 30s;
}
```
证书可以在 [sslforfree](https://www.sslforfree.com/) 之类的免费 CA 获得，当然也可以使用 OpenSSL 生成自签名的证书。配置完之后就可以通过 Nginx 愉快的玩耍 HTTP 2.0了🙈。

### 相关链接
* [Supporting HTTP/2 for Google Chrome Users](https://www.nginx.com/blog/supporting-http2-google-chrome-users/)
* [Module ngx\_http\_v2_module](http://nginx.org/en/docs/http/ngx_http_v2_module.html)
* [nginx version: nginx/1.11.6](https://www.nginx.com/resources/admin-guide/installing-nginx-open-source/)



[NPN]: http://ogovd1xl2.bkt.clouddn.com/xnpn-negotiation.png.pagespeed.ic.jL_X7qsh4g.png?imageView2/1/w/696/h/348
[ALPN]: http://ogovd1xl2.bkt.clouddn.com/how-alpn-works.png?imageView2/1/w/696/h/348
[os]: http://ogovd1xl2.bkt.clouddn.com/wwww.png?imageView2/1/w/1031/h/348
