---
layout: post
title: 同步的ajax&&jquery源代码里高大上的一段
date: 2013-11-19
categories: tech
published: true
---

###sync-ajax

遇到的公共框架的bug（异步操作返回之前不该执行的操作执行了）想要hack一下解决，想到jquery里有这样的用法：`$.ajax(url, { async: false })`，就跑去看jquery.ajax的源代码，看看jq是怎么block掉js执行的。ctrl+f了半天木有找到`while`，觉weird，觉厉。开始仔细的一点一点的看源代码，还是没看到什么地方有奇怪的trick可以block js执行。

换个思路，到MDN看了一下XmlHttpRequest的API，竟发现

{% highlight javascript %}
void open(DOMString method, DOMString url, optional boolean async, optional DOMString? user, 
optional DOMString? password);
{% endhighlight %}

无语凝咽。原来是浏览器的native实现，那就无解了，可能真只有`while(true)`了。

###jquery中的一段代码

非常清晰：

{% highlight javascript %}
// Install callbacks on deferreds
for ( i in { success: 1, error: 1, complete: 1 } ) {
	jqXHR[ i ]( s[ i ] );
}
{% endhighlight %}

确实高大上，想不到更elegant的写法了。