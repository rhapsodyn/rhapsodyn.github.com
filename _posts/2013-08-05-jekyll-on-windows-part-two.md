---
layout: post
title: 在windows上用jekyll太难了（续）
date: 2013-08-05 20:20:00
categories: tech
---
这个问题前前后后折磨了我两个星期——jekyll不能正确的build出**ol**（md to html），作为一个强迫症患者，我当然无法容忍在markdown的文档中手写`<ol></ol>`。

于是，到jekyll的github respository去提了个issue，热心的程序猿朋友礼貌地指出：这不是jekyll的issue，应该是markdown引擎的问题。

jekyll的默认markdown引擎是`maruku`，今天终于有时间了，开始一步一步的debug maruku。前前后后可能搞了40min，发现所有问题都出在这一句：

{% highlight ruby %}
	return :olist    if l =~ /^[ ]{0,1}\d+\..*\w+/
{% endhighlight %}

那个神奇的正则表达式，不能match`1. 中文`…………

算了，碰到正则表达式，果断的绕道了。

在jekyll的config中把markdown引擎改为`redcarpet`，问题解决！

1. 中
2. 文 
3. 可
4. 以
5. 用
6. 了