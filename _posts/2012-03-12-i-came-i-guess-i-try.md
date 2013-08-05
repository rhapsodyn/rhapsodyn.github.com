---
layout: post
title: I came, I guess, I try !!
categories:
- tech
tags: []
status: publish
type: post
published: true
meta:
  _edit_last: '1'
---
用nodejs+mongodb+jquery.autocomplete自己捣鼓了个suggestion。

X疼和不解的是为什么明明是连localhost有9ms的receiving-time，而访问某些远程的资源反而&lt;9ms，那个receiving-time到底是个啥东西？

![capture1](/images/QQ1.png)

想了半天，似乎不是资源大小的问题、也该不是tcp链接快慢的问题。

嗯，have a try：
{% highlight javascript %}
res.write(JSON.stringify(array));
setTimeout(function () {
	res.end();
}, 1000);
{% endhighlight %}
结果：

![capture2](/images/QQ2.png)

成就感什么的跃然之上啊！！
