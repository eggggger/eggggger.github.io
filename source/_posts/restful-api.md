---
title: 【译】RESTful APIs, 一个巨大的谎言
date: 2017-01-04 22:59:44
tags: OpenSearch
category: 写码 
---

为什么你会受益于让这个流行的范式归于平静？

<!--more-->
![RESTful API, rest in peace](https://mmikowski.github.io/images/2015-08-10-rip-small.jpg)

### 重要更新

我新增了关于  [JSON-Pure APIs](https://mmikowski.github.io/json-pure)  最佳实践的后续内容。但是在此之前，请先阅读下面的内容。

### RESTful APIs 很好用，真的吗？

如果你看过最近 10 年的互联网开发者简历或相关招聘信息，那么可以原谅你认为 **RESTful APIs** 是 Web 开发真神赐予的礼物。RESTful APIs 到处都是，甚至市场营销人员也把他们放在销售材料中推销给 CEO 和 HR.

所以 RESTful APIs  真的是一个好想法吗？在回答这个问题之前，让我们先看下 REST 的出处，以及 RESTful APIs 的定义。

### REST 的起源？

REST 流行起来是因为  [Roy Fielding](https://en.wikipedia.org/wiki/Roy_Fielding) 在他 2000 年的博士论文  [Architectural Styles and the Design of Network-based Software Architectures](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) 中介绍并推荐了它。Roy 因为他对 web 开发，特别是 HTTP 规范的贡献而闻名。

### 什么是 RESTful APIs ？

[REST](https://en.wikipedia.org/wiki/Representational_state_transfer)  即表述性状态转移，是一种用于构建可扩展 web 服务的架构风格。Roy 提倡使用他在 HTTP 标准 中参与制定的请求方法来赋予 HTTP 请求语义。

因此当使用 REST 时，下面的 HTTP 请求都有不同的语义：

*   `GET /object/list`
*   `POST /object/list`
*   `PUT /object/list`

这只是一部分 HTTP 请求方法。完整的列表如下：`CONNECT`, `DELETE`, `GET`, `HEAD`, `OPTIONS`, `PATCH`, `POST`, `PUT`, `TRACE`。可以原谅你从没有听说过其中的某些方法，因为它们中的一部分也一直都没得到客户端或服务端的支持。
Roy 同样主张使用 他参与制定的 HTTP 响应状态码来传达响应的语义。共有 38 种 HTTP 响应状态码，下面是一个完整的列表。为了节约空间，我缩短了一些响应头：

| Method | Method |
| --- | --- |
| 200 OK | 201 Created |
| 202 Accepted | 203 Not authorized |
| 204 No content | 205 Reset content |
| 206 Partial content ||
| 300 Multiple choice | 301 Moved permanently |
| 302 Found | 303 See other |
| 304 Not modified | 306 (unused) |
| 307 Temporary redirect ||
| 400 Bad request | 401 Unauthorized |
| 402 Payment required | 403 Forbidden |
| 404 Not found | 405 Method not allowed |
| 406 Not acceptable | 407 Proxy auth required |
| 408 Timeout | 409 Conflict |
| 410 Gone | 411 Length required |
| 412 Preconditions failed | 413 Request entity too large |
| 414 Requested URI too long | 415 Unsupported media |
| 416 Bad request range | 417 Expectation failed |
| 500 Server error | 501 Not implemented |
| 502 Bad gateway | 503 Service unavailable |
| 504 Gateway timeout | 505 Bad HTTP version |

因此，这种基于 HTTP 的 API 事务最少包含了以下内容：

*   the HTTP request method verb, e.g `GET`
*   请求方法动词, 如 `GET`
*   请求地址, 如 `/object/list`
*   请求实体, 如 表单字段
*   响应状态码 如 `200 OK`
*   响应实体, 如 JSON 数据

许多人在通过 HTTP 提供 web 服务时接受了这种模式，而这就是我们所说的 **RESTful API**.

由于各种原因，RESTful API 可能比上面显示的更复杂，其中一些我们会在下文中讨论。我们将忽略和网络传输及缓存有关的复杂因素，因为这些问题对所有通信渠道是通用的。

### RESTful APIs 事实上是相当糟糕的

在许多方面，例如内容交付上，REST 都是一个不错的机制，它也很好得为我们服务了二十多年。但是是时候打破沉默了，承认 RESTful API 可能是在 web 软件中广泛使用的最糟糕的想法之一。Roy 或许是一个厉害的家伙，他也的确有很多不错的想法。可是无论如何，我都不相信  RESTful APIs 是其中之一。

我们很快将看到更好的构建互联网 API 的方式，但在完全领会这个解决方案之前，我们首先得知道 RESTful APIs 的 5 个主要问题，它们使 API 昂贵，冗余，并且容易出错。让我们开始吧！

### 问题 #1: 关于 RESTful API 是什么，几乎达不成共识

你有没有注意到没有人称呼他们的 API 为 “RESTpure”？ 他们叫它 “RESTful” 或者 “RESTish”。那是因为没有人能够在全部的请求方法，实体和响应状态码的真正含义上达成一致。
设想一下，例如，我们应该用 `200 OK` 表示成功的更新了一条记录还是要用 `201 Created`？看上去我们应该用 `250 Updated`，但是这个状态码并不存在。而且谁能向我解释一下 `417 Expectation failed` 的真正含义。当然我是指除了 Roy 之外的人。

HTTP 方法和响应状态码的词汇过于模糊和不完整，无法就其语义达成一致。至少在我所知道的，没有任何管理机构能召集大家一起把事情讲清楚。一个公司定义的 `200 OK` 必然与另一家公司定义的有细微且令人讨厌的区别。这意味着一个 RESTful 模式不能像我们想要的那样可以预测。
如果只有这一个问题，我可能还会用 RESTful APIs。但 RESTful APIs 的问题层出不穷...

### 问题 #2:  REST 词汇的支持不完全

即使就 REST 的各个语义达成一致，我们还会遇到另一个实际的问题：大部分客户端和服务器程序不能完全支持 HTTP 协议 里定义的动词或响应状态码。例如，大部分浏览器对`PUT` 或 `DELETE` 仅提供有限的支持。并且很多服务端程序通常也不能正确地实现这些方法。

那么我们如何突破这些限制呢？一种常用的方法是在浏览器的表单里嵌入预期的请求方法。这意味着 REST 请求现在包含：

*   一个 HTTP 请求方法, 如 `POST`
*   一个请求地址, 如 `/object/list`
*  一个我们实际期望的方法，嵌入在请求的实体中，如  `DELETE`
*  请求实体本身，如表单字段数据

响应状态码的处理也好不到哪去。不同的 web 浏览器 (或者服务器) 对响应状态码的解释也经常各不一样。例如，如果遇到 `307 Temporary redirect` 状态码，一个浏览器可能允许客户端JavaScript 在重定向之前处理响应并取消它，另一个浏览器可能就禁止客户端 JavaScript 处理响应。所有程序中真正可靠的响应状态码只有 `200 OK` 和 `500 Internal server error`。除此之外，支持程度一个比一个糟糕。因此，我们也常常可以看到在返回的的实体里嵌入有实际想要的响应状态码。
当然，即使我们能让每个人都对什么是 REST 达成一致，并神奇地修复所有已连入互联网的程序对 REST 支持的缺陷，我们仍然还有一个问题：REST 词汇不够丰富。

### 问题 #3: REST 词汇对 APIs 来说不够丰富

REST 的方法和响应状态码太过于局限，无法有效地表示所有应用程序所需的各种请求和响应。想一下，我们创建了一个应用程序，并要向 HTTP 客户端 发送一个 “render complete” 响应。我们不能使用 HTTP 响应状态码，因为它不存在并且 HTTP 不能扩展。好吧，我想我们还是把实际预期的状态码嵌入到响应实体中去吧。

还有另外一个问题：HTTP 响应状态码 (200, 201, 202 等) 是和 HTTP 请求方法 (`GET`, `POST` 等)没有直接联系的数字，而我们的实体载荷通常是 JSON 格式。所以执行 REST 请求就好像发送一个目的地是斯瓦希里却包含英语内容的包裹，并通过打鼓声来获得交付确认。这种复杂性往往会造成巨大的困惑和错误。它给我们带来了下一个问题：调试。

### 问题 #4: RESTful APIs 非常难调试

如果你曾经用过 RESTful API，你大概知道他们几乎难以调试。这是因为我们必须查看 7 个不同的地方来整理出一个完整的事务中发生了什么：

*   HTTP 请求方法, 如 `POST`
*   请求地址, 如 `/object/list`
*   请求实体中嵌入的实际期望的方法，如 `DELETE`
*   请求实体中的实际数据，如 表单数据
*   响应状态码，如 `200 OK`
*   响应实体中嵌入的实际期望的响应状态码，如 `206 Partial content`
*   响应实体中的实际数据

我们的问题不仅有两个 REST 词汇上的限制，大家就语义达不成一致。现在还有需要查找 7 个不同的地方才可能完全理解和调试一个事务。唯一还能使这更糟的事是，REST 完全绑定到了一个协议上，而不适用于任何其他的协议。当然，这也是我们的下一个问题。

### 问题  #5: RESTful APIs 通常绑定在 HTTP 上

RESTful API 打破了良好通信的一个基本原则：消息内容应该独立于传输通道。将两者混合来迷惑读者是一个历史悠久的方法。

将 HTTP 协议 与事务的意义混合使得 RESTful API 完全不可移植。将 RESTful API 从 HTTP 迁移 到其他传输协议上需要从 7 个不同的地方解构和重组信息，我们再使用这些信息对 RESTful 请求的全部含义进行编码。

幸运的是有一个更好的解决方案，避免或减轻了几乎所有 RESTful APIs 的问题。我们将这个解决方案称为 ** JSON-Pure APIs **。

### 未来的方向： JSON-Pure APIs

[JSON-Pure APIs](https://mmikowski.github.io/the_lie/json-pure) 解决了我们刚刚讨论的大部分问题。

* 它们只使用一种传输方法来发送请求 -  如 `POST` 用于 HTTP 或 `send`  用于 WebSockets。
* 它们将请求内容与传输机制完全分离。所有的错误，警告和数据通通放在 JSON 请求实体中。
* 它们只用一个响应状态码来确认消息的正确接收 - 如 HTTP 的 `200 OK`。
* 它们将响应内容与传输机制完全分离。所有的错误，警告和数据通通放在 JSON 请求实体中。
* 它们易于调试，因为事务信息可以在使用了单个特定领域词汇的实体中的易读 JSON 里找到。
* 它们可以轻松地在传输协议中迁移和共享，如 HTTP/S，WebSockets，XMPP，telnet，SFTP，SCP 或 SSH

JSON-Pure APIs  的产生是因为开发者发现  RESTful APIs  对浏览器或开发人员不是很友好。在 API 中将信息和传输通道分离，通常会更快，并且总是更可靠，更易于使用，迁移和调试。现在，如果你使用 Twitter API，受虐狂可以选择 RESTful API，而我们其他人则可以使用 JSON-Pure API (他们叫它 “Web API”)。

在过去的十年里，我经常被要求使用 RESTful API 来代替 JSON-Pure。上一次我不得不支持一个 RESTful API 是在2011年。值得庆幸的是，服务端团队同意通过简单地将他们的 HTTP 方法 和 响应状态码 填充到 JSON 中来提供 JSON-Pure API 以及 RESTful 变体。几个月以后，我认识的每一个人使用 API 的人都切换到 JSON-Pure 版本了。这相比之前好多了。

### JSON-Pure APIs 的未来

**更新**: 我新增了一些内容来解释 [JSON-Pure APIs](https://mmikowski.github.io/json-pure) 的最佳实践的细节。当然，这些建议并不是在任何情况下都是最佳的，但它们的确被我频繁地用于需要响应 [单页 Web 应用](https://www.amazon.com/Single-Page-Applications-end---end/dp/1617290750) 的 API 类型上。一如既往，你的道路可能会有所不同。但是如果你的用例和我一样的话，你可能会想让你的下一代 API “RESTless” 并且 “go Pure”。

Cheers, Mike

Written on August 10, 2015

> 本文转载自：[众成翻译](http://www.zcfy.cc)
> 译者：[eggggger](http://www.zcfy.cc/@eggggger)
> 链接：[http://www.zcfy.cc/article/2155](http://www.zcfy.cc/article/2155)
> 原文：[https://mmikowski.github.io/the_lie/](https://mmikowski.github.io/the_lie/)
