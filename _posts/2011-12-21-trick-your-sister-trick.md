---
layout: post
title: trick your sister trick
categories:
- other
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
asp.net的control在.Net4.0之前不能设定id的模式（ClientIDMode），control在客户端生成的时候由于master等坑爹货存在id会被“污染”，直接用JQuery的`$("#id")`将选不到id。
stackoverflow上第一篇搜到的帖子有人抛了个trick出来，直接被vote到了前面成了标准答案。
其实。。。。 用`$("*[id='clientid']")`真还不如多加个class专门用来选择`$(".someAdditionalClass")`。。

[http://jsperf.com/asp-net-control-id-selector-vs-class-selector](http://jsperf.com/asp-net-control-id-selector-vs-class-selector)

ps：在.Net 4.0之前版本和JQuery 1.4版本的时候，可能是最优解了
