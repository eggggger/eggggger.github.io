---
title: OpenSearch
date: 2016-08-10 23:50:13
tags: OpenSearch
category: 写码 
---

OpenSearch 是 Amazon.com 子公司 A9 所提出的一个用来分享搜索结果的微格式集合，OpenSearch 说明文档可以描述一个搜索引擎，它能被搜索客户端识别（如 Chrome ）。

<!--more-->
## 例子
具体介绍之前，先拿百度举个🌰 

如下图所示，在 Chrome 的地址栏中输入百度的网址并按下 tab 键就能直接使用百度搜索而无须先打开百度（前提条件：之前访问过百度）

![baidu](http://7xqpu6.com1.z0.glb.clouddn.com/baidu.png)

那么怎么实现这样的效果呢？
***
首先，我们用 Chrome 的检查功能查看百度的源码，可以发现以下这行代码

![baidu link](http://7xqpu6.com1.z0.glb.clouddn.com/baidu-link.png)

这个 &lt;link&gt; 标签定义了 OpenSearch 文档的引用，其中：

* **type** 属性（必须）包含 application/opensearchdescription+xml
* **rel** 属性（必须）包含 search
* **href** 属性（必须）包含一个指向 OpenSearch 说明文档的 URL
* **title** 属性（可选）可以包含一个描述搜索引擎的文本 如 百度搜索
* **&lt;head&gt;** 标签 （可选）应该包含一个 profile 属性，其值为 http://a9.com/-/spec/opensearch/1.1/

***

然后我们再看一下百度这个 OpenSearch 文档的具体内容

[![baidu OpenSearch](http://7xqpu6.com1.z0.glb.clouddn.com/baidu-opensearch.png)](https://www.baidu.com/content-search.xml)

```
<OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/">
<ShortName>百度搜索</ShortName>
<Url type="text/html" template="https://www.baidu.com/s?wd={searchTerms}"/>
</OpenSearchDescription>
```
这个文档包含了 Chrome 所必需的最少的两个字段，其中

* **&lt;ShortName>** 元素包含了一个标识搜索引擎的标题（注意长度不能超过16个字符），它会出现在 Chrome 的地址栏里 也就是上图中的百度搜索
* **&lt;Url>** 元素包含了搜索链接的格式，它告诉客户端如何使用搜索引擎，其中
	* **template** 属性（必须）定义了URL的模板（[URL 模板语法](#OpenSearch-URL-模板语法)）
		* **searchTerms** 将被客户端（如 Chrome）替换为对应的关键词
		* **count** 将被替换为搜索结果的数量
		* **startIndex**  将被替换为第一个搜索结果的数量 （默认 indexOffset 属性的值）
		* **startPage**  将被替换为第一组搜索结果的页码 （默认 pageOffset 属性的值）
		* **language** 将被替换为期望得到的搜索结果的语言（默认 \*）
		* **inputEncoding** 将被替换为执行搜索时的编码（默认 UTF-8）
		* **outputEncoding** 将被替换为期望得到的搜索结果的编码（默认 UTF-8）
		
	* **type** 属性（必须） 定义了资源的 MIME type （如 text/html 表明搜索返回的是一个文档，Chrome 会跳转到该文档）
	* **rel** 属性（可选）定义了资源和文档的关系
		* **results** （默认）表示指定格式的搜索结果的请求
		* **suggestions** 表示指定格式的搜索建议的请求
		* **self** 表示这个说明文档的规范 URL
		* **collection** 表示一组资源的请求

	* **indexOffset** 属性（可选）定义了第一个搜索结果的起始值（默认为 1）
	* **pageOffset** 属性（可选）定义了第一组搜索结果的页码（默认为 1）

***

所以，如果你想要你的网站也支持在 Chrome 里按 tab 搜索，只需要在HTML文档里加入一个特定的 **&lt;link>** 标签指向你的OpenSearch XML文档就行了（你的OpenSearch文档里至少要包含 **&lt;ShortName>** 和 **&lt;Url>**）。
	
## OpenSearch 元素
OpenSearch 文档中除了上述提到的**&lt;ShortName>**、**&lt;Url>**元素外还有下面这些元素：

* **&lt;Description>** （必需）包含一段描述搜索引擎的文本（不多于1024个字符）

* **&lt;LongName>**（可选）包含一段标识搜索引擎的文本（少于48个字符）

* 补充中...
	
## 附

##### OpenSearch URL 模板语法

```js

    ttemplate      = tscheme ":" thier-part [ "?" tquery ] [ "#" tfragment ]
    tscheme        = *( scheme / tparameter )
    thier-part     = "//" tauthority ( tpath-abempty / tpath-absolute / tpath-rootless / path-empty )
    tauthority     = [ tuserinfo "@" ] thost [ ":" tport ]
    tuserinfo      = *( userinfo / tparameter )
    thost          = *( host / tparameter )
    tport          = *( port / tparameter )
    tpath-abempty  = *( "/" tsegment )
    tsegment       = *( segment / tparameter )
    tpath-absolute = "/" [ tsegment-nz *( "/" tsegment ) ]
    tsegment-nz    = *( segment-nz / tparameter )
    tpath-rootless = tsegment-nz *( "/" tsegment )
    tparameter     = "{" tqname [ tmodifier ] "}"
    tqname         = [ tprefix ":" ] tlname
    tprefix        = *pchar
    tlname         = *pchar
    tmodifier      = "?"
    tquery         = *( query / tparameter )
    tfragement     = *( fragement / tparameter )
```	 
