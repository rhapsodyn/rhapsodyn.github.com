---
layout: post
title: JQuery Sucks (1) —— 无节操接口升级
date: 2013-07-15 20:10:00
categories: tech
---

{% highlight javascript %}
$.getJSON(url, { num: 1 })
.then(function (data) {
	return $.getJSON(url, { num: data });
})
.then(function (data) {
	return $.getJSON(url, { num: data });
})
.done(function (data) {
	console.log(data);
});
{% endhighlight %}