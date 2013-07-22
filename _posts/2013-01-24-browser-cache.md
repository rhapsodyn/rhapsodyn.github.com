---
layout: post
title: Browser Cache
categories:
- tech
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---

###先说结论：###
1. Google最牛逼。
2. 利用浏览器缓存的最佳实践是：
	让response header里的Cache-Control：max-age=天长地久 
	给每个静态资源打上fingerprint（有proxy挡在前面就用改文件名，没proxy改querystring就够了）

**再来开始扯**
<!--more-->
###Last-Modified ETag Expire Cache-Control###
首先，他们都是http-header，ETag和Cache-Control要“潮一点”，生在http1.1时代。

last-modified和etag是一对，记录资源最后的修改时间。last-modified是个datetime所以最小可以到秒，etag是个guid所以可以到ms。

expire和cache-control是一对，用来标明资源缓存的有效时间。expire生的早，功能没那么多，据传还有各种问题（服务器、客户端时间对不上等等），但为了兼容一些奇葩浏览器（IE67什么的），很多服务器还是用了这个属性。cache-control就要牛逼多了，一堆的参数可以选择，看起来就给人很厉害的感觉，各种网站也是推荐用这个东西来控制客户端缓存（cache-control会覆盖expire）。

看，也不是多复杂的东西啊。

###到底咋缓存呢？###
上面那段的最后一句话是骗你的，其实这东西挺复杂的。

首先，你得先知道，我们想用cache-control缓存啥东西——静态资源

缓存的东西放在哪儿——本地，浏览器缓存文件夹里

为啥放在文件夹里就快了——废话，放在本地就不浏览器用向服务器发起请求了，不用三次握手不用分析header不用xxxx了

**浏览器缓存的目的在于减少请求**，一下子就觉得厉害了吧？

但事实上，没那么厉害。因为cache-control只在很少的情况下才会去检查，比如在地址栏回车，比如新打开浏览器输入地址回车（晓得为啥经常发布了以后脚本还是旧的了吗），比如在一个站点下点击链接跳来跳去。只要轻轻一下F5，你苦心经营的cache-control策略就见鬼去了（晓得为啥让用户刷新一下就正常了吗）。理论上，如果有设置cache-control，浏览器就没有请求，没有请求，你怎么更新服务器上的资源都是白瞎啊。（跟据我的瞎测试表明，当服务器的response header里没设置cache-control的时候，浏览器也会启用一些缓存策略，而缓存的过期时间根据浏览器不同可能会不同，但是总体来说——都是“很久”）

那么，不用cache，所有资源都去发起请求，是否就是世界末日了呢？

当然不是。

请求了以后，发现if-modified-since和last-modified吻合了，不是还会返回304吗？

亲，你是否忘了304的status code的全部？<strong>304 not modified</strong>！既然header返回了not modified的，什么sb浏览器还会去继续请求body啊（实事证明我又sb了，RFC就规定了304 response根本就没有body）。所以304的请求比起200来，还是要少了不少流量（可测试的，但不要傻到测localhost）（chrome终于奇葩了一回，命中缓存的状态码竟然是 200 from cache）

所以，google的最佳实践就讲的通了，1）把cache时间竟然拉长，能不请求就不请求。2）如果内容变了，那我就给你一个新的位置，你就必须来个新请求

###这就整完了？###
当然没有，我咋会这么不持久。

注意到上面的一段中请求的发起者都是浏览器了没？

问：除了浏览器还有什么能发起请求？曰：服务器！问：你是在耍我吗？曰：proxy服务器。

好吧，proxy服务器不会F5，所以proxy服务器的缓存更有效！（google说大部分proxy服务器会把只有querystring不同的请求当成同样的请求处理，根据我以前用iis ARR的经验来说，似乎是这样）

PS：以上所有内容和实验均针对于IE9 chrome20等modern browser，IE6什么的，让我们呵呵好吗

###这次真完了###
终于能码个这么多字的文章了，好感动。

还可以装模做样写个reference，更感动了。

**Reference**：

+ [https://www.varnish-software.com/static/book/HTTP.html](https://www.varnish-software.com/static/book/HTTP.html)
+ [http://www.mnot.net/cache_docs/](http://www.mnot.net/cache_docs/)
+ [http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html](http://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html)
+ [http://www.cnblogs.com/rubylouvre/archive/2012/09/04/2670073.html](http://www.cnblogs.com/rubylouvre/archive/2012/09/04/2670073.html)
