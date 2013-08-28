---
layout: post
title: JQuery Sucks (1) —— 无节操接口升级
date: 2013-07-15 20:10:00
categories: tech
---

{% highlight javascript %}
var url = '/service/double';
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

你以为上面那段代码会输出`8`是吗？我也这样以为，遗憾的是，只有JQuery1.8以后的版本才会满足你的期待。而1.8以前的版本，只会输出`2`。

why?? `then`的api[在此](http://api.jquery.com/deferred.then/)，你可以看到，这个接口在1.5 1.7都有改变过。而且，版本的演进并不是向前兼容的。		
印象中，`ajax`相关接口风格从1.4开始也有很大改变（引入promise，把url单独从setting中提出来等等，这个接口后面的文章再喷），但至少是向前兼容的。

现在的JQuery已经分裂成了1.x和2.x两条线，2.x直接就抛弃了对IE8及以前版本的支持，这应该是最没节操(大胆)的一次升级了吧。照理说，分成了两条产品线，1.9版本应该是兼容以前版本的吧？不幸的是，你还需要一个这种东西——[jQuery Migrate plugin](http://jquery.com/download/)。

总的来说——IMHO——越**普及**越**大众**的库应该在接口升级上越能兼容以前的接口。特别是JS这种动态语言，因为静态语言至少还可以在编译期报个错什么的，动态语言就悄悄的crash了啊，多吓人啊。

其实，都是程序员，向前兼容的难处，大家都懂。有些接口就是该废掉，这是没错，但是JQuery这种**世界级**的库，不是应该有更elegant的解决方案才对么？

PS：这个系列的文章将会黑的非常主观，以满足我扭曲的心理